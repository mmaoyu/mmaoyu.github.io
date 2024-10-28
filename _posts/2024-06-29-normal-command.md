---
layout: post
title:  Normal command
categories:
- article
tags:
- tools
- develop
---

### command
```
 #count group by
 awk  '{count[$7]++;} END {for(i in count) {print i " " count[i]}}' a.txt
 
 
 # iptables nat by output when ip not exist
   iptables -t nat -A OUTPUT -p tcp -d xxxx.yyyy.zzzz.oooo -j DNAT --to-destination 127.0.0.1

   #delete policy
   iptables -t nat -L -n -v --line-number
   iptables -t nat  -D OUTPUT <line-number>

 # curl 
   curl http://127.0.0.1:2020/healthCheck --connect-timeout 5 --max-time 5 -vvv

 ```

### too many openfiles
```
 # processes open files
 lsof | awk '{ print $1 " " $2; }' | sort -rn | uniq -c | sort -rn | head -15
 lsof -p <process_id>

 # system limit
 ulimit -a

```
