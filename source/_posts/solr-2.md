title: solr由浅入深-2
date: 2015-01-09 19:46:53
categories: solr
tags: ['solr', 'java', '搜索']
description:
---
基本的指令介绍,包括搜索和启动.
<!--more-->
###搜索
---
#####单字搜索
搜索foundation

~~~sh
curl "http://localhost:8983/solr/collection1/select?wt=json&indent=true&q=foundation"
~~~
限定返回的字段

~~~sh
curl "http://localhost:8983/solr/collection1/select?wt=json&indent=true&q=foundation&fl=id"
~~~
针对具体的字段进入单字的匹配

~~~sh
curl "http://localhost:8983/solr/collection1/select?wt=json&indent=true&q=name:foundation"
~~~

#####短语搜索
搜索参数q后面跟的短语一定要放在双引号中,并且单词和单词之间的空格要使用"+"号代替.

~~~sh
curl "http://localhost:8983/solr/collection1/select?wt=json&indent=true&q=\"CAS+latency\""
~~~

#####组合搜索
需要同时包含单词lucene和单词second

~~~sh
curl "http://localhost:8983/solr/collection1/select?wt=json&indent=true&q=name:(%2Blucene+%2BSecond)"
~~~

包含单词lucene并且不允许出现second

~~~sh
curl "http://localhost:8983/solr/collection1/select?wt=json&indent=true&q=name:(%2Blucene+-Second)"
~~~

包含单词lucene或者second

~~~sh
curl "http://localhost:8983/solr/collection1/select?wt=json&indent=true&q=name:(%2Blucene-%2BSecond)"
~~~

**注: '+' URL encode ==> %2B**

###启动
---
~~~sh
bin/solr start #常规启动
bin/solr start -f #foreground模式启动
bin/solr start -p 8383 #指定端口启动
bin/solr -i #查找已经启动solr实例,并显示基本信息比如版本号和内存占用
~~~

###终止
---
~~~sh
bin/solr stop
bin/solr stop -all
~~~



