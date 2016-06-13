title: JVM-debug
date: 2014-08-10 20:59:33
categories: debug
tags: ['JVM', 'debug']
description:
---
大家都有过遇到线上程序LOAD突然狂飙的场景，要排查到为何狂飙，我们当务之急就是要找到导致CPU飙升的原因。如果是进程级的应用，如Nginx、Apache等都还比较容易排查，但如果是JVM中的某个线程导致的，估计有人就要开始抓瞎了。
<!--more-->
大家都有过遇到线上程序LOAD突然狂飙的场景，要排查到为何狂飙，我们当务之急就是要找到导致CPU飙升的原因。如果是进程级的应用，如Nginx、Apache等都还比较容易排查，但如果是JVM中的某个线程导致的，估计有人就要开始抓瞎了。   
很多人都或多或少的知道有这么一个脚本，能帮你大致定位到现场导致LOAD飙升的JVM线程，脚本大概如下。
```shell
#!/bin/ksh
 
typeset top=$\{1:-10\}
typeset pid=$\{2:-$(pgrep -u $USER java)\}
typeset tmp_file=/tmp/java_$\{pid\}_$$.trace
 
$JAVA_HOME/bin/jstack $pid > $tmp_file
ps H -eo user,pid,ppid,tid,time,%cpu --sort=%cpu --no-headers\
        | tail -$top\
        | awk -v "pid=$pid" '$2==pid{print $4"\t"$6}'\
        | while read line;
do
        typeset nid=$(echo "$line"|awk '{printf("0x%x",$1)}')
        typeset cpu=$(echo "$line"|awk '{print $2}')
        awk -v "cpu=$cpu" '/nid='"$nid"'/,/^$/{print $0"\t"(isF++?"":"cpu="cpu"%");}' $tmp_file
done
 
rm -f $tmp_file
```
现在我们就来拆解其中的原理，以及说明下类似脚本的适用范围。
####步骤1：dump当前JVM线程，保存现场
```shell
$JAVA_HOME/bin/jstack $pid > $tmp_file 
```  
保存现场是相当的重要，因为问题转瞬之间就会从手中溜走（但其实LOAD的统计机制也决定了，事实也并不是那么严格）
####步骤2：找到当前CPU使用占比高的线程
```shell 
ps H -eo user,pid,ppid,tid,time,%cpu --sort=%cpu 
```
![](http://img4.tbcdn.cn/L1/461/1/b_12679_1389500395_604324404.png)  
列说明  
USER：进程归属用户  
PID：进程号  
PPID：父进程号  
TID：线程号  
%CPU：线程使用CPU占比（这里要提醒下各位，这个CPU占比是通过/proc计算得到，存在时间差）
####步骤3：合并相关信息
我们需要关注的大概是3列：PID、TID、%CPU，我们通过PS拿到了TID，可以通过进制换算10-16得到jstack出来的JVM线程号​
```shell 
typeset nid="0x"$(echo "$line"|awk '{print $1}'|xargs -I{} echo "obase=16;{}"|bc|tr 'A-Z' 'a-z') 
```
![](http://img4.tbcdn.cn/L1/461/1/b_12679_1389501427_1335323358.png) 
####适用范围说明
看似这个脚本很牛X的样子，能直接定位到最耗费CPU的线程，开发再也不用担心找不到线上最有问题的代码～但，且慢，姑且注意下输出的结果，State: WAITING 这是这个啥节奏～  
这是因为ps中的%CPU数据统计来自于/proc/stat，这个份数据并非实时的，而是取决于OS对其更新的频率，一般为1S。所以你看到的数据统计会和jstack出来的信息不一致也就是这个原因～但这份信息对持续LOAD由少数几个线程导致的问题排查还是非常给力的，因为这些固定少数几个线程会持续消耗CPU的资源，即使存在时间差，反正也都是这几个线程所导致。
