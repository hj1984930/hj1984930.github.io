---
layout: post
title: 使用Docker创建自己专属翻墙服务
description: ""
modified: 2015-10-22
tags: [docker,shadowsocks ]
image:
  feature: abstract-2.jpg

---

###使用Docker创建自己专属翻墙服务
- - -


####首先需要一台云主机

我用的是首都在线的位于美国达拉斯的云主机
系统：centos7.0-64bit
配置：4CPU 4G内存，60G硬盘

####安装docker运行的环境

~~~
curl -sSL https://get.docker.com/ | sh
service docker start
~~~

####安装shadowsocks的docker镜像

~~~
docker pull oddrationale/docker-shadowsocks
~~~

####运行shadowsocks镜像

~~~
docker run -d -p 2022:2022 oddrationale/docker-shadowsocks -s 0.0.0.0 -p 7777 -k xxxx -m aes-256-cfb
~~~

* shadowsocks的端口是7777
* -k 后面是密码
至此一个shadowsocks的docker镜像已经成功运行了，docker给我们带来的太多的方便和快捷，有木有

####本地客户端配置ss客户端
根据你的本机环境，安装shadowsocks客户端，官方的下载已经被和谐了
可以从我的[百度云盘共享下载](http://pan.baidu.com/s/1mgJ9i32)
我的是macbook，安装后，添加一个服务器配置
![vpn配置](/images/shadowsocks.png)

####享受一下把

![google](/images/google.png)
![twitter](/images/twitter.png)

速度还是很快的，10M独享带宽
