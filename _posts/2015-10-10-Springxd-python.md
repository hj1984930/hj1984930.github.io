---
layout: post
title: Springxd调用python模块计算沪深300指数
description: ""
modified: 2015-10-10
tags: [springxd, python ]
image:
  feature: abstract-10.jpg

---

###Springxd调用python模块计算沪深300指数
- - -


####python环境准备


* springxd的集群，由于python中调用了mysql数据库，所以需要安装MySQL-python模块

centos6.4中使用epel源安装以下组件，包括pip

~~~bash
yum install gcc gcc-c++ python-devel mysql-devel python-pip
~~~

然后使用pip安装MySQL-python

~~~bash
pip install MySQL-python
~~~

* 脚本权限设置

~~~
chmod 775 /opt/SpringXD.py
chown spring-xd:pivotal /opt/SpringXD.py
~~~

####springxd stream创建

* 行情数据从kafka中实时获取
* 数据经过python模块进行指数计算
* 计算结果放到kafka中，对外提供订阅服务

~~~bash
stream create --name hs300 --definition "kafka --zkconnect=10.2.29.4:2181 --topic=price --autoOffsetReset=smallest --outputType=text/plain | shell --command='/usr/bin/python /opt/SpringXD.py' | file --dir=/opt/test/ --name=hs300" --deploy
~~~






