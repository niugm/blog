---
layout: post
title: 使用Jenkins实现Hexo自动部署
date: 2018-02-16 23:42:38
tags: hexo jenkins
---
> 春节假期，趁着孩子睡觉的时候，手痒想折腾下前端自动化，于是就有了这个博客及这篇文章。

本文记录使用Jenkins自动化Hexo的部署，Jenkins是基于Java的持续集成工具，用来做Hexo的自动化部署，功能绰绰有余，这里仅为熟悉流程及配置。

### 实验环境

* 宿主主机：vultr的入门VPS，Ubuntu系统
* Web Server：Docker内运行的Nginx容器
* Jenkins：Tomcat内运行的Jenkins.war包
* 代码托管：Github

### Github配置

1. 新建一个repo，用于存放Hexo的源码，我这里是自建develop分支为工作分支。

2. 分支内settings，设置Webhooks的Payload URL：http://www.youhost.com:8080/jenkins/pluginManager/advanced

![Github Webhooks](http://p4j8u5n0f.bkt.clouddn.com/2018222145338.jpg)
注意：在github内配置web hook的payload url时，必须以/结尾

### Jenkins配置

关于Jenkins的安装，网上有很多帖子可参考，这里不做说明。
注意：启动Jenkins后，如果提示是offline运行，在地址栏直接输入地址，修改升级站点的地址 https->http，也可以网上找一个镜像站点地址，这样以后安装插件速度能快些

### 命令调试