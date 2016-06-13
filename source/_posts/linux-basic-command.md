title: linux基本命令
date: 2014-12-29 15:57:03
categories: linux
tags: ['linux', 'command', '日常']
description:
---
linux下必须要掌握的基本命令
<!--more-->
v3
```shell
chmod +x * #给当前所有脚本赋予执行权限, 使用ll查询当前权限
jps -mlv
ps -ef | grep java
netstat -anp | grep 8080
vmstat 1 30 #每秒输出一次输出30次, r运行队列(说多少个进程真的分配到CPU), b被阻塞的进程
iostat -dxk 1 1 #查看IO相关性能值
#avgqu-sz:  平均I/O队列长度。
#await:  平均每次设备I/O操作的等待时间 (毫秒)。
#svctm: 平均每次设备I/O操作的服务时间 (毫秒)。
#%util:  一秒中有百分之多少的时间用于 I/O 操作，即被io消耗的cpu百分比
#如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明I/O 队列太长，io响应太慢，则需要进行必要优化。如果avgqu-sz比较大，也表示有大量io在等待。
ifstat #查看网卡当前流量,读入(in), 出口(out)
top -Hp 20943
ps -mp 20943 -o THREAD,tid #查找最耗CPU的线程
printf "%x\n" 21742 #10进制转16进制
jstack 20943 | grep 5dee #输出指定线程的栈信息
jmap -heap 20943 #输出堆内存信息
jmap -histo:live 20943 #输出活着的对象数目. 
B  byte
C  char
D  double
F  float
I  int
J  long
Z  boolean
[  数组，如[I表示int[]
[L+类名 其他对象
jmap -dump:format=b,file=/tmp/dump.dat 21711
jstat -gc 20943 250 4 #每隔250秒取样1次.取4次
S0C    S1C    S0U    S1U      EC       EU        OC         OU       PC     PU    YGC     YGCT    FGC    FGCT     GCT  
S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）
EC、EU：Eden区容量和使用量
OC、OU：年老代容量和使用量
PC、PU：永久代容量和使用量
YGC、YGT：年轻代GC次数和GC耗时
FGC、FGCT：Full GC次数和Full GC耗时
GCT：GC总耗时 
``` 
