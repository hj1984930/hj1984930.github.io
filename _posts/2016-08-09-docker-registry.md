---
layout: post
title: 建立私有docker仓库registry
description: ""
modified: 2016-08-09
tags: [docker, centos, registry]
image:
  feature: docker.png
  
---

## 建立私有docker仓库registry

### 环境说明
* centos6.6
* docker1.7


### 建立私有registry

* docker-registry 是官方提供的工具，可以用于构建私有的镜像仓库。

* 容器运行
~~~bash
docker run -d -p 5000:5000 -v /opt/docker-registry:/tmp/registry registry
~~~

仓库建立在容器的/tmp路径下，然后指向本地的特定路径`/opt/docker-registry`

* centos上安装

~~~bash
yum install -y python-devel libevent-devel python-pip gcc xz-devel
pip install docker-registry
~~~
