---
layout: post
title:  kuberlets coredns 无法获取svc地址
categories:
- SRE
- kuberlets
tags:
- 问题排查
- dns
---

**kuberlets coredns 无法获取svc地址**

**现象**

pod内部使用svc名字无法连接，改用svc ip后可以

**检查**

- pod内部cat /etc/resolv.conf
  nameserver 169.254.25.10

- 检查节点有监听169.254.25.10:53
  netstat -tulpn | grep 53

- 尝试从节点解析域名：失败
  nslookup [svc-name].[namespace].svc.cluster.local 169.254.25.10
  
- arping 169.254.25.10 有回报包
  ip addr 中本机此地址配置了noarp，此处回包那从其他机器回来的，ip 冲突
  
**结论**
  coredns 默认ip地址 169.254.25.10 被同网段其他机器使用，导致ip冲突
