title: github初级操作(小笨蛋版本)
date: 2014-08-11 14:57:13
categories: git
tags: ['git', 'github']
description:
---
github的初级操作,适用于单人使用基本的git命令操作远程github.  
<!--more-->  
###步骤1: 登录GitHub创建一个新的repository### 
###步骤2: 将repository克隆到本地###    
找到新创建的repository, 复制SSH的URL, 打开命令行窗口输入命令    
```
git clone git@github.com:keainono/keainono.github.io.git 
```  
现在在本地有了一个映射到GitHub远程的空的repository.  
###步骤3: 操作本地repository###  
进入本地的repository, 增加或者修改一些文件  
###步骤4: 提交###  
打开命令行窗口
```
git add * #这里的*是通配符,是指所有文件,也可以换成具体的文件名
git commit -m "BALABALA..." #注意,这里引号里的备注内容不可以为空,必须要填,否则会提交失败
```
###步骤5: 提交到远程repository###
以上的操作只是在维护本地的版本,没有提交到远程的仓库,也就是说远程的仓库还是旧的,现在提交到远程
```
git push origin master #origin是远程仓库默认的别名, master指的是远程的仓库的主干, 类似于svn的trunk
```


###小笨蛋看懂了吗 ^o^ ###

