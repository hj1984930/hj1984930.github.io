---
layout: post
title: Pivotal HD 3.0的集群规划及安装配置
description: ""
modified: 2015-08-31
tags: [hadoop, pivotal, 安装, 配置]
image:
  feature: abstract-8.jpg

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

~~~ java
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

~~~ java
  if test -f /sys/kernel/mm/redhat_transparent_hugepage/defrag; then echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag; fi  
~~~

###4. 以上准备工作完成后，安装Ambari Server
* 安装httpd服务
* 创建Staging目录

~~~ shell
mkdir /staging
chmod a+rx /staging
~~~

* 从https://network.pivotal.io/products/pivotal-hd连接下载Pivotal Ambari 1.7.1压缩包，并解压缩

~~~ shell
tar -xvzf /staging/AMBARI-1.7.1-87-centos6.tar.gz -C /staging/
~~~

* 配置本地的YUM源

~~~ shell
/staging/AMBARI-1.7.1/setup_repo.sh
~~~

* 测试可以YUM可用

~~~ bash
curl http://localhost/AMBARI-1.7.1/repodata/repomd.xml
~~~

* 通过本地的YUM源安装Ambari Server

~~~ bash
yum install ambari-server
~~~

* 配置Ambari Server

~~~ bash
ambari-server setup
~~~
根据提示配置，需要Oracle JDK1.7，不支持JDK1.6，另外需要一个关系型数据库，如postgres或是mysql均可

* 启动Ambari Server

~~~ java
ambari-server start
~~~

###5. 下载需要的安装文件
| 文件名       | 下载         | 描述            |
|:------------|:-----------:|----------------:|
| Ambari-1.7.1   | [下载链接](https://network.pivotal.io/products/pivotal-hd#/releases/206)   | Ambari的sever和agent |
| PHD-3.0.1.0   | [下载链接](https://network.pivotal.io/products/pivotal-hd#/releases/206)   | Pivotal Hadood套件包括HDFS, YARN, HBASE, HIVE, OOZIE, ZOOKEEPER. |
| PADS-1.3.1.0   | [下载链接](https://network.pivotal.io/products/pivotal-hd#/releases/206)   | Pivotal高级功能组件，包括HAWQ, PXF, MADlib. |
| PHD-UTILS-1.1.0.20   | [下载链接](https://network.pivotal.io/products/pivotal-hd#/releases/206)   | 工具包，包括监控报警等，如 Ganglia, Nagios  |

* 下载后的文件解压缩至staging目录：

~~~ java
tar -xzf /tmp/{stack}.tar.gz -C /staging/
~~~

* 对于以上四个安装包，每一个都要配置YUM

~~~ bash
/staging/{stack}/setup_repo.sh
~~~

### 6. 登录到Ambari Server的管理台

默认地址是 http://ip:8080
默认用户名密码是 admin/admin

1. 创建一个新的集群
2. 修改YUM的配置为本地的YUM（就是刚刚前面自己配置的）
3. 编辑集群的主机名和SSH KEY
4. 选择要安装的软件
5. 分配Masters和Slaves
6. 后台程序会自动检测目前环境下有哪些不符合要求的，会给出解决方法，一定要手动都处理完
7. 最后就是等待了，要安装十几分钟的
8. 至此一个新的Hadoop集群已经创建完成了

![Pivotal Hadoop Cluster](/images/ambari-server.png)




