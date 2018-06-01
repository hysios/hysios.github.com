---
layout: post
title: "awscn code-deploy 手动部署到自有的服务器当中"
date: 2017-09-08 18:51
comments: true
categories: 
---

CodeDeploy 终于进入中国市场了，我正好需要部署一下新的项目，拿来练下手

CodeDeploy 已经更新了版本，我都不太会用了。
CodeDeploy 是 AWS 出品的一套代码部署的方案，与 Elastic Beanstalk 类似功能的产品，两者功能上差异不大，因为是两个不同的小组，分别进行开发的，也是通过不同的角度去实践，自动化部署的方案的结果吧。

1. CodeDeploy 偏向是比较传统的 script 部署，优点是灵活编写脚本，缺点，相对于繁琐一点
2. EB 集成化一般语言的基础进行部署（C, Java, Php, Python, Ruby) 等，指对特定语言或框架很便捷，但是呢扩展相对比较麻烦一些

两者各有利弊，这里简单介绍一下 CodeDeploy 部署需要注意的问题，当然本身 CodeDeploy 有一个向导模式，你可以简单的使用这个方式去部署项目，非常方便，但是如果你有特定需求，像我一样需要做一些调整
那么这里的经验，可以给你一些借鉴。

## 简单部署
CodeDeploy 部署到服务器，只需要简单三部

1. 建立部署配置（向导）
2. Push 需要部署文件到指定的 s3, git 等。
3. 触发部署任务

是不是很简单，记住这个简单的大纲，部署只需要完成这三个步骤，通过理解这个行为，让在接下来的时候，遇到问题时有一个参照的基准。

## 手动部署
来到了我们的正题，如果通过上面向导的方案，基本上部署很是简单的事情，但是往往我们会有一些特定的需求，
比如，应用到原有的预留机器上。加入旧的 LB 或 Scale Group等, 而我这里正好要部署到我的一台旧机器上, (我不想开新的机器，旧的机器还有资源可以用），步骤如下：

一、准备机器的配置部分

1. 安装 codedeploy-agent 至 ec2 instance
2. 创建 CodeDeploy Role 与 Permissions 
3. 创建 Instance Role 附加到 Role
4. 应用 Instance Role 到原存在的 Ec2 Instance, 这里需要生效的话，一定要重启 codedeploy-agent  
	在 你的 Ec2 instance ssh 输入  
	```
	sudo service codedeploy-agent restart
	```
5. 

二、项目配置部分

1. write a appspec.yml
2. 准备好 start.sh 与 stop.sh
3. nginx config

三、push 代码或binary to s3

1. 复制需要部署的代码或程序到 build 目录（必须包括 appspec.yml)
2. aws push 指定 --source build 目录。
3. aws deploy 








