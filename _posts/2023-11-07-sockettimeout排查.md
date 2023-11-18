---
layout: post
title:  一次SocketTimeout问题排查
categories:
- SRE
  tags:
- 问题排查
- socket timeout
- tcp
---

**日志中长连接上的处理出现sockettimeout 异常**

***

- ![1_issue.jpeg](/images/sre/tcp/20231107/1_issue.jpeg)

- 出现异常是偶发性的

排查思路
> 服务是否负载高
> - 统计请求量，服务负载并不高

> 服务是否被hang住
> - 每隔5秒 pstack <进程号> ，未发现异常代码堆栈

> 抓包
> - tcpdump -i <网卡名> -w issue.pcap
> 包中有大量的retransmission 包，包被重传了
> ![p1.jpeg](/images/sre/tcp/20231107/p1.jpeg)

> 包被重传的场景
> - 网络拥塞
> - 网络状况不良，丢包
> - 等等

#### 在排查一筹莫展的时候，客户仍出了一条关键信息：网络中的防火墙会将空闲超4分钟的连接断开，并且不会发送rst包，如图：
> ![n1.jpeg](/images/sre/tcp/20231107/n1.jpg)

> 结论
> 
> server设置的keepalive最大时长是10分钟，防火墙上的空闲阈值先达到，连接被断开，client此时不会知道连接被断开
此时还持有着连接，并且发送数据

> 解决方案
> 
> - 将server keepalive时长改短，小于4分钟
> - 修改系统tcp keepalive时长，默认2小时，改为小于4分钟(选择了修改客户端的此方案)
> 修改net.ipv4.tcp_keepalive，如下图
    ![n1.jpeg](/images/sre/tcp/20231107/n2.jpg)

#### 问题并未得到解决

> 排查keepalive是否生效
> - sysctl -p 
> - 抓包，没有找到发送keepalive包，但是在数据连接发送了keepalive包
    ![r1.jpeg](/images/sre/tcp/20231107/r1.jpeg)
> - 排查代码，```IOReactorConfig.soKeepAlive``` 未设置为true,keepalive可被系统设置，也能被进程设置。
> - 设置进程的keepalive后，问题最终解决

#### 最终问题解决