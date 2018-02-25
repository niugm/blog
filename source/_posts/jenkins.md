---
layout: post
title: 使用Jenkins实现Hexo自动部署
date: 2018-02-16 23:42:38
tags: hexo jenkins
---
> 春节假期，手痒想折腾下前端自动化，于是就有了这个博客及这篇文章。

本文记录使用Jenkins自动化Hexo的部署，Jenkins是基于Java的持续集成工具，用来做Hexo的自动化部署，功能绰绰有余，这里仅为熟悉流程及配置。

### 实验环境

* 宿主主机：vultr的入门VPS，Ubuntu系统
* Web Server：Docker内运行的Nginx容器
* Jenkins：Tomcat内运行的Jenkins.war包  （Ver.2.107）
* 代码托管：Github

<!-- more -->

### Github配置

1. 新建一个repo，用于存放Hexo的源码，我这里是新建develop分支为写作分支。

2. 分支内settings，设置Webhooks的Payload URL：http://www.youhost.com:8080/jenkins/github-webhook/  

![Github Webhooks](http://p4j8u5n0f.bkt.clouddn.com/2018222145338.jpg)
注意：在github内配置web hook的Payload URL时，必须以“/”结尾。

3. 进入Personal access tokens，新建一个token，勾选repo、admin:repo_hook权限，将得到的token字符串保存下来，一会Jenkins内设置要用到。

### Jenkins配置

关于Jenkins的安装，网上有很多帖子可参考，这里不做说明。
有一点注意事项：启动Jenkins后，需要先安装插件，安装系统推荐的即可。
如果在安装插件前提示offline运行,可直接进入插件设置页面http://www.youhost.com:8080/jenkins/pluginManager/advanced，修改升级站点的地址 https->http，然后地址栏输入http://www.youhost.com:8080/jenkins/restart，待Jenkins重启后，可继续之前的插件安装步骤。

插件安装完毕后，先设置Credentials，即Jenkins调用GithubAPI的授权

![2018225233435](http://p4j8u5n0f.bkt.clouddn.com/2018225233435.png)
新建一个
![2018225233144](http://p4j8u5n0f.bkt.clouddn.com/2018225233144.jpg)

Kind选择 secret text，Secret就是我们在Github设置中第3步得到的token字符串。

保存后，进入系统管理->系统设置
![2018225233858](http://p4j8u5n0f.bkt.clouddn.com/2018225233858.jpg)

这里设置github插件，Github个人用户，API URL按系统默认的即可。Credentials处，选择刚才添加的token，勾选 Manage hooks，测试通过后保存。

----

以上步骤完成后，就可以开始新建Jenkins的构建任务了。

----

1. 新建一个任务，命名为blog，任务类型：构建一个自由风格的软件项目![20182252336](http://p4j8u5n0f.bkt.clouddn.com/20182252336.png)

2. 基础设置

![201822523747](http://p4j8u5n0f.bkt.clouddn.com/201822523747.png)

设置历史构建保留的天数及个数，减少空间占用。

3. 源码管理

![2018225231021](http://p4j8u5n0f.bkt.clouddn.com/2018225231021.png)

依次填写repo地址、分支名、源码浏览器选择githubweb，URL等同于repo地址。新增一个Additional Behaviours，选择 Check out to a sub-directory，这里填写一个相对目录(Jenkins的workspace)地址  github/source

4. 选择构建触发器

![2018225231655](http://p4j8u5n0f.bkt.clouddn.com/2018225231655.png)

构建触发器的作用是当我们提交代码至Github时，Github的钩子会post数据至Jenkins的API，Jenkins获取最新代码后，运行构建命令，这样就实现了自动构建。

5. 构建

![构建选择 Execute shell](http://p4j8u5n0f.bkt.clouddn.com/2018225231924.png)

Jenkins自动运行脚本，省去人工重复输入。我这里使用了Docker， /srv/nginx/html 是容器内nginx映射的根目录，大家可以根据自己的情况自行设置。

----
以上是Jenkins任务的配置说明，配置完成后，可以在本地Hexo内新建POST，然后push到远程分支，正常情况下，Jenkins自动获取最新源码并执行构建命令，进入任务详情页，可以看到构建历史

![2018225234732](http://p4j8u5n0f.bkt.clouddn.com/2018225234732.jpg)

成功的构建是蓝色，失败的是红色。点击编号，进入构建历史详情页面，可以查看控制台输出记录，以便查找错误。