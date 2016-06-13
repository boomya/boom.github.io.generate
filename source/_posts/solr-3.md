title: solr由浅入深-3
date: 2015-01-09 19:47:00
categories: solr
tags: ['solr', 'java', '搜索']
description:
---
主要介绍了如何cloud模式启动Solr
<!--more-->
###SolrCloud
---
#####以cloud模式启动solr

~~~sh
bin/solr -e cloud #开启一段交互式的对话,通过以下几步使用内置的zookeeper启动solr集群
#1. 需要启动几个节点
#2. 指定每个节点的端口号
#3. 创建一个新的collection
#4. 指定分片数量. 如果不确定,建议使用2
#5. 指定每个分片的备份数量. 如果不确定,建议使用2
#6. 指定配置文件的目录. 有两种类型default和schemaless,default是example/solr/collection1/conf. schemaless是example/example-schemaless/solr/collection1/conf. default基本上包含的solr全部的配置.如果正在设计schema,需要实验一些特性的时候schemaless很有用. 

$ bin/solr restart -c -p 8983 -d node1 #重启端口为8983的node1节点(使用内置zookeeper)
$ bin/solr restart -c -p 7574 -d node2 -z localhost:9983 #重启node2节点,使用独立zookeeper
~~~

###场景1:在同一台机器启动2个分片的集群
---
#####执行步骤:
>1. 启动第一个节点,并同时启动内置的zookeeper服务器
>2. 启动其他剩余节点,并指向已经启动的zookeeper服务器

#####操作流程:
启动第一个节点:

~~~sh
cd node1
java -DzkRun -DnumShards=2 -Dbootstrap_confdir=./solr/collection1/conf -Dcollection.configName=myconf -jar start.jar
~~~
**-DzkRun** 启动内置的zookeeper服务器.如果是产品环境推荐使用独立的zookeeper服务器,使用*zkHost=<ZooKeeper Host:Port>*替换*zkRun*
**-DnumShards** 指定分片的数量
**-Dbootstrap_confdir** 指定配置文件的目录
**-Dcollection.configName** 指定存储到zookeeper服务器的配置信息的节点名字
**note:** *-DnumShards, -Dbootstrap_confdir 和 -Dcollection.configName*只需要在第一次启动时指定,如果后面再运行会从zookeeper加载

启动其他节点:

~~~sh
cd node2
java -Djetty.port=7574 -DzkHost=localhost:9983 -jar start.jar
~~~
**-Djetty.port** 指定jetty端口
**-DzkHost** 指定zookeeper的ip,内置zookeeper的端口是solr的端口加1000.

验证SolrCloud启动是否成功,访问http://localhost:8983/solr/#/~cloud.
![SolrCloud](http://boomya-files.qiniudn.com/solrcloud-1.png)

插入数据

~~~sh
java -jar post.jar mem.xml
~~~

搜索

~~~sh
http://localhost:8983/solr/collection1/select?q=*:*
http://localhost:7574/solr/collection1/select?q=*:*
~~~
这两个URL返回的搜索结果的数量是相同的.
搜索的过程是会搜索每一个分片,再将每个分片的结果组合返回.
**note:** 如果带上参数*distrib=false*只会返回指定分片的结果

###场景2:使用外部zookeeper启动SolrCloud集群
---
#####执行步骤:
>1. 启动外部zookeeper服务
>2. 启动SolrCloud并指向外部zookeeper服务器

#####操作流程:
**note:**使用外部zookeeper服务之前,需要删除节点目录下solr/zoo_data文件夹
启动第一个节点:

~~~sh
cd node1
java -DnumShards=2 -Dbootstrap_confdir=./solr/collection1/conf \
   -Dcollection.configName=myconf \
   -DzkHost=localhost:2181,localhost:2182,localhost:2183 \
   -jar start.jar
~~~
**note:**参数的顺序是有要求的.DzkHost必须要在zookeeper相关参数的后面.

启动其他节点:

~~~sh
cd node2
java -Djetty.port=8900 -DzkRun -DnumShards=2 \
    -DzkHost=localhost:2181,localhost:2182,localhost:2183 -jar start.jar
~~~
验证SolrCloud启动是否成功,访问http://localhost:8983/solr/#/~cloud.

