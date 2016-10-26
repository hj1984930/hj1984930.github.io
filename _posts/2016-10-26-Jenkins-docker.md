---
layout: post
title: 应用容器化及DevOps流程实践
description: ""
modified: 2016-10-26
tags: [Docker, Jenkins, Play, JDK7]
image:
  feature: jenkins.png
  
---

### 应用容器化及持续集成持续部署实践

#### 说明

* 应用基于playframework开发框架开发
* 使用git/stash管理版本

#### 目标

* 将应用容器化打包
* 使用Jenkins配置持续集成持续交付流程
* 并自动的发布到容器云平台

#### 环境准备

* 首先准备三台Centos7的虚机

| 主机名         | IP            | 运行服务|
| ------------- |:-------------:| ------:|
| T-cicd-01     | 10.2.36.39    | DCE-Contoller  |
| T-cicd-02     | 10.2.36.40    | DCE-node1,jenkins-master  | 
| T-cicd-03     | 10.2.36.41    | DCE-node2,jenkins-slave  |

#### 将play的运行环境打包成容器

* 在T-cici-03上，build用户下

* JDK1.7 play1.2.5
* 使用自己创建的rpm包

* 编写dockerfile

~~~
FROM centos:7

# Setup Java Environment
ADD jdk1.7.0_79 /usr/local/

# Add play framework
COPY play-1.2.5-xxb.el6.x86_64.rpm /tmp/

RUN rpm -ivh /tmp/play-1.2.5-xxb.el6.x86_64.rpm

# Set ENVs
RUN ln -s /opt/play/play/play /usr/local/bin/play
~~~

* docker打包

~~~bash
docker build -t sufe/play:jdk7 .
docker tag sufe/play:jdk7 10.2.36.39/sufe/play-jdk7:latest
docker push 10.2.36.39/sufe/play-jdk7:latest
~~~

~~~bash
[build@T-cicd-03 play-base-image]$ docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
10.2.36.39/sufe/sanzhu           latest              3d067f4c3db2        41 hours ago        759 MB
sufe/sanzhu                      test                3d067f4c3db2        41 hours ago        759 MB
10.2.36.39/sufe/play-jdk7        latest              81f351165aad        2 days ago          676.3 MB
sufe/play                        jdk7                81f351165aad        2 days ago          676.3 MB
daocloud.io/daocloud/dce-agent   1.4.0               d21211f327ad        4 weeks ago         227.2 MB
daocloud.io/daocloud/dce         1.4.0               574d587fd399        4 weeks ago         27.32 MB
daocloud.io/daocloud/dce         latest              574d587fd399        4 weeks ago         27.32 MB
daocloud.io/daocloud/dce-lb      1.4.0               6d6fca0b522e        4 weeks ago         65.98 MB
daocloud.io/jenkins              latest              19b1ef41dab5        6 weeks ago         714.5 MB
centos                           7                   980e0e4c79ec        7 weeks ago         196.7 MB
centos                           latest              980e0e4c79ec        7 weeks ago         196.7 MB
~~~

#### Jenkins配置CICD流程

* 创建一个任务SANZHU-Manager
* 创建三个子任务，test-env-compile-job,test-env-deploy-sanzhu-job,test-env-regression-test-job

![项目管理任务](/images/jenkins-sanzhu.png)

![编译与构建镜像](/images/jenkins-compile-1.png)

![编译与构建镜像](/images/jenkins-compile-2.png)

![编译与构建镜像](/images/jenkins-compile-3.png)

![编译与构建镜像](/images/jenkins-compile-4.png)

![部署到容器云](/images/jenkins-deploy-1.png)

![部署到容器云](/images/jenkins-deploy-2.png)

![部署到容器云](/images/jenkins-deploy-3.png)

![回归测试](/images/jenkins-test.png)


#### 执行任务看看效果

![应用交付](/images/dce-1.png)

![应用交付](/images/dce-2.png)

![应用交付](/images/dce-3.png)











