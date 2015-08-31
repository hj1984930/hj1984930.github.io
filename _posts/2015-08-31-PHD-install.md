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

~~~ css
> mkdir -p /etc/sysctl.d
> ( cat > /etc/sysctl.d/99-hadoop-ipv6.conf <<-'EOF'
## Disabled ipv6
## Provided by Ambari Bootstrap
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOF
    )
> sysctl -e -p /etc/sysctl.d/99-hadoop-ipv6.conf
~~~

* 关闭Transparent Huge Pages (THP)，添加以下内容到`/etc/rc.local`,然后重启OS

~~~ shell
  
  if test -f /sys/kernel/mm/redhat_transparent_hugepage/defrag; then echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag; fi
  
~~~
###4. 以上准备工作完成后，安装Ambari Server
* 安装httpd服务
* 创建Staging目录

~~~ shell
~> mkdir /staging
~> chmod a+rx /staging
~~~
* 从https://network.pivotal.io/products/pivotal-hd连接下载Pivotal Ambari 1.7.1压缩包，并解压缩

~~~ shell
~> tar -xvzf /staging/AMBARI-1.7.1-87-centos6.tar.gz -C /staging/
~~~
* 配置本地的YUM源

~~~ shell
~> /staging/AMBARI-1.7.1/setup_repo.sh
~~~
* 测试可以YUM可用

~~~ shell
~> curl http://localhost/AMBARI-1.7.1/repodata/repomd.xml
~~~
*通过本地的YUM源安装Ambari Server

~~~ shell
~> yum install ambari-server
~~~

*配置Ambari Server

~~~ shell
~> ambari-server setup
~~~
根据提示配置，需要Oracle JDK1.7，不支持JDK1.6，另外需要一个关系型数据库，如postgres或是mysql均可
*启动Ambari Server
~~~ shell
~> ambari-server start
~~~

