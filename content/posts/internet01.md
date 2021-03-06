---
title: "网络传输组成要素"
date: 2018-08-16T19:46:00+08:00
tags: ['tcp/ip', '网络']
draft: false
---
## 网络的组成要素
1. 中继器
>中继器是在OSI模型的第一层--物理层面上延长网络的设备。由电缆传过来的电信号或光信号经由中继器的波形调整和放大再传给另一个电缆。它无法判断数据的错误，不能再传输速度不同的媒介之间转发。
*中继集线器指可以连接多个端口的中继器，也成为中继集线器或集线器。*
2. 网桥/2层交换机
> 1. 网桥是在OSI模型的第2层--数据链路层面上连接两个网络设备。它能识别数据链路层中的数据帧，并将这些数据帧暂时存储于内存，在重新生成信号作为一个全新的帧转发给相连的另一个网段。
> 2. 由于能够存储这些数据帧，故能够链接传输速率完全不同的数据链路，并且不限制连接网段的个数。此外数据链路的数据帧中有一个FCS数据位，可用于校验数据是否正确送往目的地。网桥通过检查这个域的值，讲那些损坏的数据丢弃，从而避免发送给其他的网段。
>3. 此外，网桥还能通过MAC地址自学机制和过滤功能控制网络流量。
有些网桥能够判断是否将数据报文发送给相邻的网段，该网桥又称为自学式网桥。它会记住曾经通过自己转发的所有数据帧的MAC地址，并保存到内存表中，下次直接将数据帧发送到对应网段。
*以太网等网络中经常使用的交换集线器，现在基本也属于网桥中的一种。交换集线器中连接电缆的每个端口都能提供类似网桥的功能。*

3. 路由器/3层交换机
> 路由器是在OSI模型的第3层--网络层面上连接连接两个网络、并对分组报文进行转发的设备。网桥是根据MAC地址进行处理的，而路由器则是根据IP地址进行处理的。路由器可以连接两个不同的数据链路。路由器还有分担网络负荷的作用，甚至有些路由器具备一定的网络安全功能。
4.  4~7层交换机
>4~7层交换机负责处理OSI模型中从传输层至应用层的数据。如果用TCP/IP分层模型表述，该交换机处理以TCP等协议的传输层及其上的应用层为基础，分析收发数据，并对其进行特定的处理。
>1. 例如并发访问量非常大的一个企业级Web站点，使用一台服务器不足以满足前端的访问需求，此时需要架设多台服务器来分担。这些服务器前端的访问入口地址通常只有一个。为了能通过同一个URL将前端访问分发到后台多个服务器，可以在这些服务器的前端加上一个负载均衡器。这种负载均衡器就是4~7层交换机的一种。
>2. 实际通信中，在网络比较拥堵时，希望优先处理像语音这类对及时性要求较高的通信请求，放缓像邮件或数据转发等稍有延迟也并无大碍的通信请求。这种处理被称为带宽控制，也是4~7层交换机的重要功能之一。此外还有广域网加速器、特殊应用访问加速以及防火墙等。
5.  网关
>网关是OSI参考模型中负责将从传输层到应用层的数据进行转换和转发的设备。网关不仅转发数据还负责对数据进行转换，它通常会使用一个表示层或应用层网关，在两个不能直接通信的协议之间进行翻译，最终实现两者之间的通信。
>1. 典型的例子就是互联网邮件与手机邮件之间的转换服务。手机邮件有可能会与互联网邮件互不兼容，这是由于它们在表示层与应用层中的“电子邮件协议”互不相同所导致的。在手机邮件服务器与互联网邮件之间会设置一个邮件网关，用于它们之间的转换。
>2. 为了控制网络流量以及出于安全考虑，有时会使用代理服务器(Proxy Server).这种代理服务器也是网关的一种，称为应用网关。有了代理服务器，客户端与服务器之间无需再网络层上直接通信，而是从传输层到应用层对数据和访问进行各种控制和处理。防火墙就是一款通过网关通信，针对不同应用提高安全性的产品。
