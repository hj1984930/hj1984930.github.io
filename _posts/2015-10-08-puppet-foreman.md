---
layout: post
title: 自动化运维工具puppet和foreman使用
description: ""
modified: 2015-10-08
tags: [puppet, 自动化运维, foreman ]
image:
  feature: abstract-9.jpg

---

###自动化运维工具puppet和foreman使用经验
- - -

2013年启动和实施，稳定有效的运行了两年多

####项目背景

* 传统的运维方式，效率低下，容易出错，不利于伸缩
* 自动化的运维方式能够有效的提高工作效率，降低人工出错的可能
* 能够快速响应用户需求，短时间内完成系统环境准备和配置管理

####项目目标

* 基于开源的自动化配置和部署软件puppet建立云环境下的自动配置管理平台，能够快速的响应业务需求，集中管理和控制服务器资源
* 帮助系统管理员管理IT系统架构的整个生命周期，从环境准备到配置管理到补丁管理和应用
* 自动化重复任务，快速部署关键应用，主动的管理变化。帮助IT环境标准化、流程化、自动化、规模化

####平台架构

* puppetmaster和foreman分开部署
* puppetmaster：puppet、puppet-master、factor、foreman-proxy、puppetCA
* foreman：foreman、foreman database（mysql）、puppet

![平台架构图](/images/puppet-foreman.png)

####安装

* 使用foreman-installer安装
两台主机上安装foreman-installer安装环境

~~~bash
yum -y install http://yum.theforeman.org/releases/1.3/el6/x86_64/foreman-release.rpm
rpm -ivh http://yum.puppetlabs.com/el/6/products/i386/puppetlabs-release-6-7.noarch.rpm
rpm -ivh http://mirrors.yun-idc.com/epel/6/i386/epel-release-6-8.noarch.rpm
yum -y install foreman-installer
~~~

foreman安装

~~~bash
foreman-installer \
  --puppet-server=false \
  --foreman-proxy-puppetrun=false \
  --foreman-proxy-puppetca=false \
  --foreman-db-type=mysql
~~~

puppetmaster安装

~~~bash
foreman-installer \
  --no-enable-foreman \
  --enable-puppet \
  --puppet-server-foreman-url=https://foreman.xxb.cn \
  --enable-foreman-proxy \
  --foreman-proxy-tftp=false \
  --foreman-proxy-foreman-base-url=https://foreman.xxb.cn
~~~

####多环境部署
* 根据需要可部署多套环境，生产环境、测试环境及开发环境
![foreman界面](/images/foreman-ui.png)

####自建yum本地库
* tomcat6和tomcat7的环境使用自己打包的rpm安装

####module
* 实现base、yum源的管理、ntp时间的同步、dns的管理、sysctl的管理、ssh的配置、iptables管理、apache安装、jre6,7安装、mysql安装、tomcat6,7安装、php安装、play1.2.5安装、zabbix agent的安装、puppet agent、mysql、logstash、logrotate等

![git库中的puppet-module](/images/git-puppet.png)

####心得
* 期初modules的是自己编写，后来觉得没必要重复制造轮子，就使用forge和puppetlab上的现有资源，经过简单的修改直接使用
* 开发调试还是要经过开发环境和测试环境的验证后，在推送到生产环境中使用
* 公网上的yum源存在不稳定因素，会影响配置的推送和安装部署，导致失败，重要配置使用自己本地yum源，如果有需要，如tomcat环境，自己打包rpm，并配置yum源


