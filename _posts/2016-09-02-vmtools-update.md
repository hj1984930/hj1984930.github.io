---
layout: post
title: CentOS升级openssh至最新版
description: ""
modified: 2016-09-02
tags: [centos, openssh,]
image:
  feature: openssh.png
  
---

### CentOS升级openssh至最新版

~~~bash
mv /etc/ssh /etc/ssh.bak
yum -y install gcc* make perl pam pam-devel zlib zlib-devel openssl-devel
wget http://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-7.1p1.tar.gz
tar zxvf openssh-7.1p1.tar.gz
cd openssh-7.1p1
yum -y remove systemtap-client git openssh
vi version.h
./configure --prefix=/usr --sysconfdir=/etc/ssh --with-pam --with-zlib --with-md5-passwords
make && make install && make clean
cp contrib/redhat/sshd.init /etc/init.d/sshd
chkconfig --add sshd
service sshd start
~~~

~~~
ssh -V
OpenSSH_7.1p1, OpenSSL 1.0.1e-fips 11 Feb 2013
~~~

### 编辑/etc/ssh/sshd_config添加以下内容
`PermitRootLogin yes`
 
### 重启
~~~
service sshd restart
~~~
