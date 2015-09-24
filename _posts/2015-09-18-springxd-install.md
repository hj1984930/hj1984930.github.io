---
layout: post
title: SpringXD集群的分布式安装和部署
description: ""
modified: 2015-09-18
tags: [springxd, 大数据, 流计算, 配置]
image:
  feature: abstract-6.jpg

---

###SpringXD集群的分布式安装和部署

####1. 安装springxd
首先需要jdk7，下载rpm安装包

~~~bash
wget https://repo.spring.io/libs-release-local/org/springframework/xd/spring-xd/1.2.0.RELEASE/spring-xd-1.2.0.RELEASE-1.noarch.rpm
rpm -ivh spring-xd-1.2.0.RELEASE-1.noarch.rpm
~~~

####2. 安装mysql-server,并创建database

~~~bash
yum install mysql-server -y

mysql> create database springxd;
Query OK, 1 row affected (0.00 sec)
mysql> grant all privileges on springxd.* to springxd@'%' identified by 'springxd';
Query OK, 0 rows affected (0.01 sec)
~~~


####3. 安装redis数据库

~~~bash
yum install redis
service redis start
chkconfig redis on
~~~
修改`/etc/redis.conf`中的bind 0.0.0.0，重启redis服务

####4. 配置hadoop版本
修改`/etc/sysconfig/spring-xd`的hadoop版本

~~~
# The Hadoop distribution to be used for HDFS access
# [hadoop26 | phd21 | phd30 | cdh5 | hdp22]
HADOOP_DISTRO=phd30
~~~

####5. 配置springxd
修改`/opt/pivotal/spring-xd/xd/config/servers.yml`配置文件里的内容
添加mysql配置，redis配置，hadoop配置，zookeeper配置以及JMX配置
mysql配置：

~~~yml
spring:
  datasource:
    url: jdbc:mysql://gfxd1.xxb.cn:3306/springxd_new
    username: springxd_new
    password: springxd_new
    driverClassName: com.mysql.jdbc.Driver
~~~
redis配置：

~~~yml
spring:
  redis:
   port: 6379
   host: phd3-m1.xxb.cn
~~~
hadoop配置：

~~~yml
spring:
  hadoop:
    fsUri: hdfs://phd3-m1.xxb.cn:8020
    resourceManagerHost: phd3-m1.xxb.cn
~~~
zookeeper配置：

~~~yml
zk:
  namespace: xd
  client:
     connect: 10.2.29.4:2181,10.2.29.5:2181,10.2.29.6:2181
     sessionTimeout: 60000
     connectionTimeout: 30000
     initialRetryWait: 1000
     retryMaxAttempts: 3
~~~
JMX配置：

~~~yml
# Config to enable/disable JMX/jolokia endpoints
XD_JMX_ENABLED: true
endpoints:
  jolokia:
    enabled: ${XD_JMX_ENABLED:true}
  jmx:
    enabled: ${XD_JMX_ENABLED:true}
    uniqueNames: true
~~~


####6. 在集群中一个节点启动spring-xd-admin


~~~bash
service spring-xd-admin start
~~~

####7. 在集群的所有节点启动spring-xd-container

~~~bash
service spring-xd-container start
~~~

####8. 启动后可以通过web管理界面查看

可以看到container中有两台
![springxd-admin-ui](/images/springxd-admin-ui.png)



