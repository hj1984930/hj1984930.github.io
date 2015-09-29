---
layout: post
title: Networker配置部署客户端
description: ""
modified: 2015-09-29
tags: [networker, 客户端, 备份 ]
image:
  feature: abstract-3.jpg

---

###Networker配置部署客户端
安装文件存放在`ddbackup`服务器的`/emcsoftware`下面

###1. 安装linux客户端

~~~
yum install lgtoclnt-8.1.1.8-1.x86_64.rpm  -y
~~~


###2. 修改hosts文件
修改`/etc/hosts`，添加以下内容

~~~
10.2.28.236     DD2500.xxb.cn   DD2500
10.2.28.238     ddbackup.xxb.cn ddbackup
~~~

###3. 在networker管理台配置客户端
* 新增客户端向导
![新增客户端向导](/images/client-new-1.png)
* 指定客户端名称
![指定客户端名称](/images/client-new-2.png)
* 选择文件系统备份
![选择文件系统备份](/images/client-new-3.png)
* 选择备份目标池
![选择备份目标池](/images/client-new-4.png)
* 设置备份策略和备份计划
![设置备份策略和备份计划](/images/client-new-5.png)
* 设置备份组
![设置备份组](/images/client-new-6.png)
* 设置备份目标
![设置备份目标](/images/client-new-7.png)
* 完成配置
![完成配置](/images/client-new-8.png)

###4. networker客户端无法连接处理

* networker相关服务重启

~~~bash
/etc/init.d/gst stop
/etc/init.d/gst start
service networker stop
service networker start
~~~

* vba相关服务重启

~~~
emwebapp.sh --stop
emwebapp.sh --start
~~~