title: solr由浅入深-1  
date: 2015-01-03 21:22:27  
categories: solr  
tags: ['solr', 'java', '搜索']  
description:  
---
solr由浅入深系列-1  
搭建最简solr环境
<!--more-->
# solr搭建说明

### 概要
---  
solr可以单机部署也可以结合zookeeper实现分布式集群(solrCloud). solr内嵌了zookeeper,在启动solr的同时也会启动zookeeper,使用内嵌的zookeeper有几个缺点,1.集群中增加新的节点时需要同时增加zookeeper, 2.当某个节点挂掉需要重启时,同时也会重启zookeeper,造成了集群的不稳定.所以推荐使用独立的zookeeper集群,会在部署过程中麻烦些,不过提高了集群的稳定性.

### 运行(简单集群)
---
run:

~~~sh
bin/solr start -e cloud -noprompt
~~~
终端显示  

~~~sh
Welcome to the SolrCloud example!
Starting up 2 Solr nodes for your example SolrCloud cluster.
...
Started Solr server on port 8983 (pid=8404). Happy searching!
...
Started Solr server on port 7574 (pid=8549). Happy searching!
...
SolrCloud example running, please visit http://localhost:8983/solr
~~~
自动生成2个index
![shard](http://lucene.apache.org/solr/assets/images/quickstart-solrcloud.png)

###数据导入
---
run:

~~~sh
java -classpath dist/solr-core-4.10.3.jar -Dauto -Drecursive org.apache.solr.util.SimplePostTool docs/
~~~
*SimplePostTool*是solr提供的一个支持多种文件格式的导入工具.

~~~sh
Posting files to base url http://localhost:8983/solr/update..
Entering auto mode. File endings considered are xml,json,csv,pdf,doc,docx,ppt,pptx,xls,xlsx,odt,odp,ods,ott,otp,ots,rtf,htm,html,txt,log
Entering recursive mode, max depth=999, delay=0s
Indexing directory docs (3 files, depth=0)
POSTing file index.html (text/html)
POSTing file SYSTEM_REQUIREMENTS.html (text/html)
POSTing file tutorial.html (text/html)
Indexing directory docs/changes (1 files, depth=1)
POSTing file Changes.html (text/html)
Indexing directory docs/solr-analysis-extras (8 files, depth=1)
...
2945 files indexed.
COMMITting Solr index changes to http://localhost:8983/solr/update..
~~~
导入并索引了2945个doc.

###简单查询
---
~~~sh
http://localhost:8983/solr/collection1/select?q=solr&wt=json&indent=true
~~~
