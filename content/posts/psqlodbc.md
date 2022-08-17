---
title: "UTF8字符解析错误PG执行失败"
date: 2022-08-17T20:25:01+08:00
tags: ['unixODBC', 'psqlODBC', 'SQLBindParameter']
mermaid: true
draft: false
---
最近在团队项目中发现，应用在使用unixodbc bind方式执行insert语句时报错：
```
ERROR: invalid byte sequence for encoding “UTF8“: 0x08
```

遇到这个错误，最开始我们以为是客户端或服务端字符编码不对，因此在pg服务端查看

```bash
#PG数据库服务端编码查询
show server_encoding
```

```bash
#PG数据库客户端编码查询
show client_encoding
```

发现服务端为UTF8，客户端为SQL-ASCII，于是查看官方文档，使用

```bash
export PGCLIENTENCODING=UTF8
```
这时通过psql连接数据库后查看到的编码已经为UTF8了。但是问题并没有解决。

此时只好重新梳理代码调用栈。
如下是项目应用的调用栈，Application为应用代码，DbApi为基于Poco的数据库应用抽象API，unixODBC数据库驱动管理器为访问PG数据库，使用psqlODBC数据库驱动。

```mermaid
flowchart LR
Appication --> DbApi --> unixODBC --> psqlODBC
```
应用层使用拼sql的方式执行语句是没有问题的。这里的功能，应用层使用bind参数绑定方式执行sql，这对于对某个表批量插入数据来说，效率更好。首先准备sql语句
```sql
insert int table_r(no, name, age) value ($1, $2, $3)
```
经过for循环，批量对参数值进行绑定。如此仅一次insert模版，后续仅传送绑定参数即可，节约了与PG服务器通信的带宽。

理清流程后，逐步在Application层、DbApi层进行了绑定参数值的打印，发现并没有出现乱码。
由于unixODBC和psqlODBC都是so形式存在，源代码需要下载。因此先绕过数据库驱动，在PG服务端查看日志。PG服务端日志默认在pg_log下。使用psql登陆服务端执行如下语句打开详细日志
```sql
alter system set log_statement to 'all';
select pg_reload_conf();
```
查看日志发现发送到服务端的绑定参数已经乱码。由此可推测数据应该是在客户端代码部分出错的。

unixODBC只是驱动管理器，提供了通用的数据库操作API，但是具体实现还是psqlODBC。因此考虑查看psqlODBC日志。

官网上psqlODBC介绍了两种日志：MyLog和CommLog，但是并没有具体的说明。结合psqlODBC源码，需要设置odbc.ini中特定数据库的参数(0为关闭；1为打开；2为详细日志)
```bash
[CM_DB]
...
Debug=2
CommLog=2
```
如此就能在/tmp下看到mylog_process_202201203.log和commlog_process_20221203.log的日志文件了。
有了日志文件，再结合源码，就能帮助我们更好的定位问题。

最后，我们定位到，数据绑定过程中，引用入参被传入到一个临时的string类型变量中。但是绑定和sql执行是分为2个函数。当绑定函数结束时，这个临时变量会被释放。在执行时，临时变量在内存中的值变成乱码了。因此导致执行时字符乱码。解决问题后，sql语句能正常执行了。






