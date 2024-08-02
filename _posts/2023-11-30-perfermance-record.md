---
layout: post
title:  性能采集
categories:
- SRE
tags:
- 问题排查
- cpu
- io
- 性能
---

**性能采集**
---

> 系统io/cpu数据采集，输出到当前目录的iostat.log
>
> `iostat -dx 5 |tee iostat.log.$(date '+%Y_%m_%d_%H_%M_%S')`
>
> cpu内存数据采集，输出到当前目录的vmstat.log
>
>`vmstat 5 |tee  vmstat.log.$(date '+%Y_%m_%d_%H_%M_%S')`
>
> 进程数据采集，输出到当前目录的pidstat.log
>
>`pidstat -du 5 | tee pidstat.log.$(date '+%Y_%m_%d_%H_%M_%S')`

> 日志解释
>- iostat.log
>
>#### 每隔5秒，输出磁盘io统计信息，信息解释如下（也可以使用man iostat查看每一列的解释）
>
> ![p_iostat_1.png](/images/sre/perfermance/p_iostat_1.png)
>
> 磁盘并发数计算：
>
>concurrency=(r/s+w/s) * (svctm/1000)
>
> 一般需要计算%util 为100%时的并发数，如果并发数为1，即为磁盘串行，raid的并发数会好
>
>如果多磁盘并发数=1，需要考虑文件系统串行问题
>
>- vmstat.log
>
>每隔5秒输出，信息解释如下（也可以使用man vmstat查看每一列的解释）：
> ![p_vmstat_1.png](/images/sre/perfermance/p_vmstat_1.png)
>
#### 查询swap占用进程
```
 overall=0
for status_file in /proc/[0-9]*/status; do
    swap_mem=$(grep VmSwap "$status_file" | awk '{ print $2 }')
    if [ "$swap_mem" ] && [ "$swap_mem" -gt 0 ]; then
        pid=$(grep Tgid "$status_file" | awk '{ print $2 }')
        name=$(grep Name "$status_file" | awk '{ print $2 }')
        printf "%s\t%s\t%s KB\n" "$pid" "$name" "$swap_mem"
    fi
    overall=$((overall+swap_mem))
done
printf "Total Swapped Memory: %14u KB\n" $overall
```

### 结论
cpu密集型机器：
- vmstat
  1. us 列高或者sy高,优化代码
  2. si，sy超过20%需要关注，系统高，使用perf排查
  3. 大部分情况，r列也会有进程排队时间
  4. cs > 100,000
  5. iostat
%util 磁盘使用率一般比较小

io密集型机器：
- vmstat
  1. b列显示很多
  2. wa 也很高
  3. iostat
%util的值比较大（此时对于读请求大可能是致命的，读必须等到拿数据，写没有这么致命可以合并缓冲）

内存交换严重
- vmstat
  1. si 和so有高值
- 修改swapiness
  ```
  sudo sysctl vm.swappiness=1
  vi /etc/sysctl.conf
  vm.swappiness=1
  sysctl -p
  ```
磁盘结论：
- 传统盘< ssd
- 传统盘：raid 10，缓存配置writeBack策略，大部分可以运行良好
- ssd：明显提升

其他建议：
- xfs文件系统
- swapiness/硬盘队列调度器设置恰当的值

推荐阅读
- https://plantegg.github.io/2018/01/23/10+%E5%80%8D%E6%80%A7%E8%83%BD%E6%8F%90%E5%8D%87%E5%85%A8%E8%BF%87%E7%A8%8B/