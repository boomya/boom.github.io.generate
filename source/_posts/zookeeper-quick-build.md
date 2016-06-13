title: zookeeper快速搭建
date: 2015-01-14 21:18:11 
categories: 集群
tags: ['zookeeper', '集群'] 
description:
---
快速搭建zookeeper集群,简单介绍了搭建过程,基本指令和日志的配置.
<!--more-->
###standalone模式
---
#####1. 创建配置文件conf/zoo.cfg
~~~sh
tickTime=2000
dataDir=/var/zookeeper
clientPort=2181
~~~
**tickTime**单位是毫秒.这是zookeeper的最小时间单位,心跳间隔时间,同步超时时间都是基于这个值.
**dataDir**保存zookeeper保存在内存中数据的快照,必须指定一个已经存在的目录.
**clientPort**监听客户端连接的端口.

#####2. 启动zookeeper
~~~sh
bin/zkServer.sh start zoo.cfg #指定加载zoo.cfg配置文件,可以不指定,默认是zoo.cfg
~~~

#####3. 停止zookeeper
~~~sh
bin/zkServer.sh stop zoo.cfg
~~~

###replicated模式
---  
#####1. 更新配置文件zoo.cfg  
~~~sh
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/var/zookeeper/zookeeper-1
# the directory where transaction log is stored.
# this parameter provides dedicated log device for ZooKeeper
dataLogDir=/var/zookeeper/zookeeper-1/log
# the port at which the clients will connect
clientPort=2181

server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2889:3889
server.3=127.0.0.1:2890:3890
~~~
**initLimit**连接集群中的leader实例的超时时间. initLimit=10的意思是10个tickTime时间.
**syncLimit**与leader实例同步数据的超时时间.
**dataLogDir**用于单独设置transaction log的目录,transaction log分离可以避免和普通log还有快照的竞争.
**server.x=[hostname]:[port1]:[port2]**在dataDir下创建myid文件,将x写入.port1用于连接leader和其他节点.port2用于选举leader.
#####2. 启动多个zookeeper
每个zookeeper节点都需要有一个对应的配置文件.    

~~~sh
bin/zkServer.sh start zoo-1.cfg
bin/zkServer.sh start zoo-2.cfg
bin/zkServer.sh start zoo-3.cfg
~~~

###基本命令介绍
---
#####1. 连接zookeeper节点

~~~sh
bin/zkCli.sh -server 127.0.0.1:2181
~~~
#####2. 基本指令

~~~sh
ls / #查看根目录下的key
create /zk_test my_data #创建一个新的key-value
get /zk_test #获取指定key的value
set /zk_test my_data #设置
delete /zk_test #删除key
~~~

###日志配置
---
配置DailyRollingFile类型的日志,在conf/log4j.properties中增加以下配置

~~~sh
# Define some default values that can be overridden by system properties
zookeeper.root.logger=INFO, CONSOLE, DAILYROLLINGFILE
zookeeper.console.threshold=INFO
zookeeper.log.dir=.
zookeeper.log.file=zookeeper_rolling.log
zookeeper.log.dailyFile=zookeeper_daily.log
# Add DAILYROLLINGFILE to rootLogger to get log file output
log4j.appender.DAILYROLLINGFILE=org.apache.log4j.DailyRollingFileAppender
log4j.appender.DAILYROLLINGFILE.File=${zookeeper.log.dir}/${zookeeper.log.dailyFile}
log4j.appender.DAILYROLLINGFILE.DatePatter='.'yyyy-MM-dd
log4j.appender.DAILYROLLINGFILE.layout=org.apache.log4j.PatternLayout
log4j.appender.DAILYROLLINGFILE.ConversionPattern=%d{ISO8601} [myid:%X{myid}] - %-5p [%t:%C{1}@%L] - %m%
~~~
修改bin/zkEnv.sh,找到以下代码

~~~sh
if [ "x${ZOO_LOG4J_PROP}" = "x" ]
then
    ZOO_LOG4J_PROP="INFO,ROLLINGFILE"
fi
~~~
修改为

~~~sh
if [ "x${ZOO_LOG4J_PROP}" = "x" ]
then
    ZOO_LOG4J_PROP="INFO,CONSOLE,DAILYROLLINGFILE"
fi
~~~
再重新启动zookeeper,新的日志配置才会生效.

###JVM参数调整
---
在conf目录下新建java.env文件,增加以下内容

~~~sh
#!/usr/bin/env bash
export JVMFLAGS="-Xms256m -Xmx256m"
~~~
重新启动zookeeper.




