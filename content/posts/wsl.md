---
title: win10 wsl
date: 2021-01-20 21:45:13 
tags: [wsl]
draft: false
---
windows wsl可以很方便的让我们在win10中运行各种发行版的linux。但是在使用wsl之后，我安装了很多软件，直接导致在wsl ubuntu中再使用apt来安装软件后，C盘直接变红了。

## 解决办法
在网上查找资料，使用LxRunOffline,可以将wsl从C盘转移到其他盘。果断使用起来。
1. 下载LxRunOffline
[LxRunOffline](https://github.com/DDoSolitary/LxRunOffline)
2. 查看当前wsl的发行版
```powershell
./LxRunOffline l
Ubuntu
```
3. 移动到D盘wsl目录中
```powershell
.\LxRunOffline move -n Debain -d D:\wsl

``` 

经过以上操作后，C盘终于回蓝，同时D盘多了wsl目录。

## 日常问题
1. 可能系统重新开机后，ubuntu子系统打不开
解决办法：管理员用户打开powershell
```powershell
netsh winsock reset
```

