---
layout: post
title: Pivotal HD 3.0的集群规划及安装配置
description: ""
modified: 2015-08-31
tags: [hadoop, pivotal, 安装, 配置]
image:
  feature: abstract-9.jpg

---

###Pivotal HD 3.0的集群规划及安装配置

####1. 准备工作
* 操作系统：CentOS 6.4+ (64-bit)
* 软件工具：python2.6以上、Oracle jdk-7u67-linux-x64.tar.gz、openssl-1.0.1e-16.el6.x86_64以上
* 数据库：mysql5.x

####2. 操作系统无密码认证
这个大家都会，就不说了

####3. 其他一些注意事项
* 所有服务器的时钟一定要同步，一致
* 主机名的dns解析
* iptables关闭
* selinux关闭
* 如果安装了PackageKit，要修改配置文件`/etc/yum/pluginconf.d/refresh-packagekit.conf`，改为**enabled=0**
* 关闭ipv6