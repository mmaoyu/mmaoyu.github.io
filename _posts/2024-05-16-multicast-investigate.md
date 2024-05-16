---
layout: post
title:  Multicast investigate
categories:
- SRE
tags:
- troubleshooting
- keepalive
- multicast
---

**Multicast investigate**
---
We discovered that a keepalive VIP exist on every host when we use VVRP and multicast. However, When swtich to uticast everything functions correctly.

Keepalive utilizes multicast to notify other hosts of floating IP infomation, why isn't the multicast infomation being handled correctly?

The first question comes to mind is :
1. Are multicast packages being sent? The following shows the multicast packages being sent.
```
[root@192-168-0-200 ~]# tcpdump -i ens192 -c 30 host 224.0.0.18
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
10:48:48.319765 IP 192-168-0-200 > vrrp.mcast.net: VRRPv2, Advertisement, vrid 51, prio 152, authtype simple, intvl 1s, length 20
10:48:49.320918 IP 192-168-0-200 > vrrp.mcast.net: VRRPv2, Advertisement, vrid 51, prio 152, authtype simple, intvl 1s, length 20
```
1. So are multicast packages being received? The previous command indicates that no packets are being received.
2. Given that these machines are purchased from a cloud vendor, the network may have limitations on sending multicast packets. We conduct a iperf test to receive multicast packets. Which also showed no multicast packets being received. 
   ```
   1.  Start multicast listener:
       iperf -s -u -B 224.0.55.55 -i 1
   2. Start multicast server:
        iperf -c 224.0.55.55 -u -T 32 -t 3 -i 1
   ```

It appears that the issue lines in the network or the cloud platform. it was discovered that Aliyun cloud provides a manual detailing the steps for creating and managing a multicast network:
https://help.aliyun.com/zh/cen/user-guide/create-and-manage-a-multicast-network . According to the manual, multicast network funcationality is tured off by default. Therefore , it is necessary to create and enable the multicast network before utilizing it. You can refer to the provided link for detailed instructions on how to create and manage a multicast network on the Aliyun cloud platform. Other cloud platform might have similar default setting, requiring users to manually create and enable it. 



Addition information, keepalive config:
```
! Configuration File for keepalived
global_defs {
   router_id "rb02"
   vrrp_strict
}

vrrp_script check_web {                      
    script "check.sh"        
    interval 2                                 
    weight 2                                   
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0                        
    virtual_router_id 51                          
    priority 100
    advert_int 1
    #nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
#    unicast config
#    unicast_src_ip 192.168.0.204
#    unicast_peer {
#        192.168.0.200           
#    }
    virtual_ipaddress {
       192.168.0.99/24 dev eth0 label eth0:1     ###VIP,按需修改，所有节点使用同一个
    }
    track_script {
        check_web # 执行定义的脚本
    }
}

```
