title: git-flow
date: 2016-02-01 20:53:27
categories: git
tags: ['git', 'git flow']
description:
---
以git-flow为基准, 设计一套符合初创公司的git多人协作最佳实践.
<!--more-->
## 分支开发流程
#### 1. 了解Git, Maven的基本用法, 了解git-flow
- ##### Git   
![image](http://backlogtool.com/git-guide/cn/img/post/intro/capture_intro1_2_2.png)
- ##### Git 常用命令
```shell
#拉取远程仓库主干版本代码
git clone git@112.124.125.71:server/dk-ext-share.git dk-ext-share
#拉取远程仓库dev分支代码
git clone -b dev git@112.124.125.71:server/dk-ext-share.git dk-ext-share-dev
#更新并提交本地仓库
git add pom.xml
git commit -m "update pom.xml"
#拉取远程仓库
git pull
#将本地仓库提交
git push
#基于master创建dev分支, 并切换到dev分支
git checkout -b dev master
#与远程仓库同步本地分支索引
git fetch -p
#列出所有分支
git branch -a
#删除分支
git branch -d feature-t
#将当前分支与指定分支feature-t1合并
git rebase --no-ff feature-t1
#如果在合并过程中发生冲突, 解决后, 继续执行合并
git rebase --continue
#查看当前的版本状态, 包括是否有未commit, 未push, 冲突是否解决等  
git status  
```  

- ##### git-flow
![image](http://7q5a09.com1.z0.glb.clouddn.com/o_git-workflow-release-cycle-3release.png)  
1. ==master分支==被保护且锁定, 不允许做任何的提交操作, 只允许在GitLab上提交合并, 由==管理员==执行合并.  
2. ==dev分支==基于master分支创建, 用于和feature分支合并, 做上线前的联调.  
3. ==release分支==基于dev分支创建,  被保护且锁定, 不允许做任何的提交操作, 只允许在GitLab上提交合并, 由管理员执行合并. 只用于==正式发布==.
4. ==feature分支==基于dev分支创建, 是临时性分支, 在每次正式发布后需要删除. 本地测试通过后, 和==dev分支==合并. 命名规则按照当前项目的版本号来, 比如, 版本号为==1.2.2-SNAPSHOT==, 则分支名为==feature-1.2.2-SNAPSHOT==  

#### 2. 具体的操作流程
master=管理员, developer=开发人员
- ##### 开发前置准备工作 ==(developer)==  
    本地基于dev分支创建临时性功能feature分支, 并推送到远程代码仓库.
    ```shell
    #新版本号为分支名
    git checkout -b xxx-SNAPSHOT dev && git push origin xxx-SNAPSHOT:xxx-SNAPSHOT
    ```  
切换到新分支, 修改pom.xml中版本号和依赖的版本号为新版本号

- ##### 开发阶段 ==(developer)==
    在临时性功能分支下开发, 目前只能在本地测试
- ##### 联调阶段 ==(developer, master)==
    切换到dev分支, 和功能分支合并, 解决冲突后, 提交并更新远程代码仓库
    ```shell  
    git checkout dev  
    git rebase --no-ff xxx-SNAPSHOT  
    ```    
- ##### 正式发布  ==(developer, master)==
    由developer在GitLab发起merge request, dev分支合并到release分支, 以及dev分支合并到master分支, 由master在GitLab中执行合并.  
    合并后, 由master执行发布流程, 二方库需要deploy到nexus.
- ##### 正式发布后置  ==(developer, master)==
    分别在本地和GitLab删除临时性功能分支, 并且由master在GitLab中创建本次发布的tag.
