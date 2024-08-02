---
layout: post
title:  服务接口响应慢
categories:
- api
- sre
tags:
- 问题排查
- api
- 响应
---

**服务接口响应慢**

- 客户反馈接口请求响应时间慢
- 请求处理数没有锐增

***排查步骤***
1. 健康检查接口是否都正常
2. 是否有大量日志产生（按大小切割）
3.  查access log ，里面有响应时间，在倒数第二列，是毫秒，服务access路径：`xxx-access.log`，nginx：`/var/log/nginx/access.log`，查询响应时间大于2秒的请求倒序：`awk '$(NF-1)>2000' filepath`
4.  java服务是否持续gc，查看gc时间：`cat xxx.gc.log |grep Failure|cut -d' ' -f 9|sort -n`，解决办法：调整jvm
5.  任务处理是否超额，具体业务具体分析，常见的方式是业务metrics，或者数据库sql
6.  mysql是否慢查询，查看慢查询日志。是否需要数据库调优
7.  docker部署的查看资源限制，`docker stats`
8.  磁盘速度最好大于50m/s，`dd bs=64k count=4k if=/dev/zero of=test oflag=dsync`，如果磁盘速度不达标，但是客户原因无法提速，调高日志输出级别到error
9.  客户是否为虚拟机部署，查看宿主机的资源，是否存在宿主机cpu或者可用内存全部分配给虚拟机的情况，如果有，调小虚拟机资源，至少给宿主机预留4核cpu，4g以上内存
10. `free -m` 确认swap使用较
11. 系统负载情况
12. 服务连接情况
