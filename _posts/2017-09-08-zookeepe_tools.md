---
layout: post
title:  zookeeper 查看工具
categories:
- zookeeper
tags:
- 分布式
---
1. 分析事务日志
```sh
java -cp zookeeper-3.4.9.jar:lib/slf4j-api-1.6.1.jar org.apache.zookeeper.server.LogFormatter /data/zookeeper/version-2/log.700000001
```

2. 分析快照日志
```sh
java -cp zookeeper-3.4.9.jar:lib/slf4j-api-1.6.1.jar org.apache.zookeeper.server.SnapshotFormatter /data/zookeeper/version-2/snapshot.3000000ec
```






