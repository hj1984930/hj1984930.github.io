---
layout: post
title: centos配置docker运行环境
description: ""
modified: 2016-08-02
tags: [docker, centos,]
image:
  feature: docker.png
  
---

## centos配置docker运行环境

### 环境说明
* centos6.6
* docker1.7

### 配置过程

添加yum源

~~~bash
tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
~~~

安装docker1.7.1

~~~bash
yum install -y docker-engine
~~~

启动docker服务

~~~bash
chkconfig docker on
service docker start
~~~

获取镜像ubuntu

~~~bash
docker pull ubuntu:12.04
~~~

列出镜像

~~~bash
docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                          12.04               93932704ad15        10 days ago         139.3 MB
centos                          latest              7322fbe74aa5        13 months ago       172.2 MB
dl.dockerpool.com:5000/mysql    latest              3c6d7e5c8c1b        21 months ago       235.6 MB
dl.dockerpool.com:5000/java     7u65-jdk            bd8bd16075a0        21 months ago       562.7 MB
dl.dockerpool.com:5000/centos   latest              87e5b6b3ccc1        22 months ago       224 MB
~~~


制作私有镜像

~~~bash
yum -y install febootstrap
febootstrap -i bash -i wget -i yum -i iputils -i iproute -i man -i vim -i openssh-server -i openssh-clients -i tar -i gzip  centos6 centos6.8-image http://mirrors.aliyun.com/centos/6.8/os/x86_64/
~~~

centos6:OS版本。

centos6.8-image:镜像文件保存到当前路径下的centos6.8-image文件夹下。

http://mirrors.aliyun.com/centos/6.8/os/x86_64/:centos6.8系统镜像路径。

导入镜像文件

~~~bash
[root@hj-t centos6.8-image]# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  sbin  selinux  srv  sys  tmp  usr  var
[root@hj-t centos6.8-image]# cd ..
[root@hj-t ~]# cd centos6.8-image/ && tar -c .|docker import - centos6.8-base
fe95b256d2793f16fe41bd7a85b86ebef0fb5bb8a1b58d72cda2ac18e5b83735
~~~

查看制作的镜像

~~~bash
[root@hj-t centos6.8-image]# docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
centos6.8-base                  latest              fe95b256d279        About a minute ago   399.2 MB
trusty                          latest              8b40146fae0a        4 days ago           0 B
<none>                          <none>              38b969ffff81        5 days ago           172.2 MB
ubuntu                          12.04               93932704ad15        2 weeks ago          139.3 MB
centos                          latest              7322fbe74aa5        13 months ago        172.2 MB
dl.dockerpool.com:5000/mysql    latest              3c6d7e5c8c1b        21 months ago        235.6 MB
dl.dockerpool.com:5000/java     7u65-jdk            bd8bd16075a0        21 months ago        562.7 MB
dl.dockerpool.com:5000/centos   latest              87e5b6b3ccc1        22 months ago        224 MB
~~~

使用刚刚制作的镜像
并修改镜像

~~~bash
[root@hj-t centos6.8-image]# docker run -t -i centos6.8-base /bin/bash
bash-4.1#
bash-4.1#
bash-4.1#
bash-4.1# cat /etc/redhat-release
CentOS release 6.8 (Final)
bash-4.1# tee /etc/yum.repos.d/xxb.repo <<-'EOF'
> [xxb]
> name=XXB Customized packages
> proxy=_none_
> enabled=1
> baseurl=http://yum.xxb.cn
> gpgcheck=0
> EOF
[xxb]
name=XXB Customized packages
proxy=_none_
enabled=1
baseurl=http://yum.xxb.cn
gpgcheck=0
bash-4.1#
bash-4.1# cat /etc/yum.repos.d/xxb.repo
[xxb]
name=XXB Customized packages
proxy=_none_
enabled=1
baseurl=http://yum.xxb.cn
gpgcheck=0
bash-4.1# rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
Retrieving http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
warning: /var/tmp/rpm-tmp.Q2TNYz: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Preparing...                ########################################### [100%]
   1:epel-release           ########################################### [100%]
bash-4.1# rpm -ivh http://yum.puppetlabs.com/el/6/products/i386/puppetlabs-release-6-7.noarch.rpm
Retrieving http://yum.puppetlabs.com/el/6/products/i386/puppetlabs-release-6-7.noarch.rpm
warning: /var/tmp/rpm-tmp.PDozFT: Header V4 RSA/SHA1 Signature, key ID 4bd6ec30: NOKEY
Preparing...                ########################################### [100%]
   1:puppetlabs-release     ########################################### [100%]
   bash-4.1# ls -l /etc/yum.repos.d/
total 40
-rw-r--r-- 1 root root 1991 May 18 15:47 CentOS-Base.repo
-rw-r--r-- 1 root root  647 May 18 15:47 CentOS-Debuginfo.repo
-rw-r--r-- 1 root root  630 May 18 15:47 CentOS-Media.repo
-rw-r--r-- 1 root root 6259 May 18 15:47 CentOS-Vault.repo
-rw-r--r-- 1 root root  289 May 18 15:47 CentOS-fasttrack.repo
-rw-r--r-- 1 root root 1056 Nov  4  2012 epel-testing.repo
-rw-r--r-- 1 root root  957 Nov  4  2012 epel.repo
-rw-r--r-- 1 root root 1250 Apr 12  2013 puppetlabs.repo
-rw-r--r-- 1 root root   95 Aug  8 03:23 xxb.repo
bash-4.1# exit
exit
You have mail in /var/spool/mail/root
~~~

提交修改后的容器

~~~bash
[root@hj-t centos6.8-image]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
ef6ed34185e4        centos6.8-base      "/bin/bash"         About a minute ago   Up About a minute                       furious_brown
[root@hj-t centos6.8-image]# docker commit -m="add epel/xxb/puppet yum repos" -a="HuangJie" ef6ed34185e4 xxb/centos6.8-image
f9e5479d080972711bbb049a21f268eaa48ff8797851824b726bb62d6d90e599
You have mail in /var/spool/mail/root
[root@hj-t centos6.8-image]# docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
xxb/centos6.8-image             latest              f9e5479d0809        20 seconds ago      399.2 MB
centos6.8-base                  latest              fe95b256d279        22 minutes ago      399.2 MB
trusty                          latest              8b40146fae0a        5 days ago          0 B
<none>                          <none>              38b969ffff81        5 days ago          172.2 MB
ubuntu                          12.04               93932704ad15        2 weeks ago         139.3 MB
centos                          latest              7322fbe74aa5        13 months ago       172.2 MB
dl.dockerpool.com:5000/mysql    latest              3c6d7e5c8c1b        21 months ago       235.6 MB
dl.dockerpool.com:5000/java     7u65-jdk            bd8bd16075a0        21 months ago       562.7 MB
dl.dockerpool.com:5000/centos   latest              87e5b6b3ccc1        22 months ago       224 MB
~~~


编写dockerfile

~~~bash
# centos tomcat7-java7
FROM xxb/centos6.8-image
MAINTAINER Docker HuangJie <huangjie@sufe.edu.cn>
RUN echo -ne "[xxb]\nname=XXB Customized packages\nproxy=_none_\nenabled=1\nbaseurl=http://yum.xxb.cn\ngpgcheck=0\n" > /etc/yum.repos.d/xxb.repo
RUN rpm -ivh http://yum.puppetlabs.com/el/6/products/i386/puppetlabs-release-6-7.noarch.rpm
RUN rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
RUN yum install java-1.7.0-sun -y
RUN yum install tomcat7-7.0.52 -y
~~~

使用dockerbuild构建一个新的镜像

~~~bash
[root@hj-t docker]# docker build -t xxb/centos6.8-tomcat7 .
Sending build context to Docker daemon 14.85 kB
Sending build context to Docker daemon
Step 0 : FROM xxb/centos6.8-image
 ---> 04a72f961cee
Step 1 : MAINTAINER Docker HuangJie <huangjie@sufe.edu.cn>
 ---> Running in 87b006753029
 ---> ed75666269b8
Removing intermediate container 87b006753029
Step 2 : RUN echo -ne "[xxb]\nname=XXB Customized packages\nproxy=_none_\nenabled=1\nbaseurl=http://yum.xxb.cn\ngpgcheck=0\n" > /etc/yum.repos.d/xxb.repo
 ---> Running in b04ea1c52941
 ---> a1d12004c223
Removing intermediate container b04ea1c52941
Step 3 : RUN rpm -ivh http://yum.puppetlabs.com/el/6/products/i386/puppetlabs-release-6-7.noarch.rpm
 ---> Running in ef5725f352a9
warning: /var/tmp/rpm-tmp.umEdxA: Header V4 RSA/SHA1 Signature, key ID 4bd6ec30: NOKEY
Retrieving http://yum.puppetlabs.com/el/6/products/i386/puppetlabs-release-6-7.noarch.rpm
Preparing...                ##################################################
puppetlabs-release          ##################################################
 ---> afdc3f858610
Removing intermediate container ef5725f352a9
Step 4 : RUN rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
 ---> Running in b20c53211e2d
warning: /var/tmp/rpm-tmp.DOY63e: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Retrieving http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
Preparing...                ##################################################
epel-release                ##################################################
 ---> 8ec75e6205cd
Removing intermediate container b20c53211e2d
Step 5 : RUN yum install java-1.7.0-sun -y
 ---> Running in bf089eca1658
Loaded plugins: fastestmirror
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package java-1.7.0-sun.x86_64 1:1.7.0.25-xxb.el6 will be installed
--> Processing Dependency: unixODBC for package: 1:java-1.7.0-sun-1.7.0.25-xxb.el6.x86_64
--> Processing Dependency: jpackage-utils for package: 1:java-1.7.0-sun-1.7.0.25-xxb.el6.x86_64
--> Running transaction check
---> Package jpackage-utils.noarch 0:1.7.5-3.16.el6 will be installed
---> Package unixODBC.x86_64 0:2.2.14-14.el6 will be installed
--> Processing Dependency: libltdl.so.7()(64bit) for package: unixODBC-2.2.14-14.el6.x86_64
--> Running transaction check
---> Package libtool-ltdl.x86_64 0:2.2.6-15.5.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package              Arch         Version                     Repository  Size
================================================================================
Installing:
 java-1.7.0-sun       x86_64       1:1.7.0.25-xxb.el6          xxb        100 M
Installing for dependencies:
 jpackage-utils       noarch       1.7.5-3.16.el6              base        60 k
 libtool-ltdl         x86_64       2.2.6-15.5.el6              base        44 k
 unixODBC             x86_64       2.2.14-14.el6               base       378 k

Transaction Summary
================================================================================
Install       4 Package(s)

Total download size: 100 M
Installed size: 163 M
Downloading Packages:
--------------------------------------------------------------------------------
Total                                            18 MB/s | 100 MB     00:05
warning: rpmts_HdrFromFdno: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
Importing GPG key 0xC105B9DE:
 Userid : CentOS-6 Key (CentOS 6 Official Signing Key) <centos-6-key@centos.org>
 Package: centos-release-6-8.el6.centos.12.3.x86_64 (@febootstrap/$releasever)
 From   : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
Warning: RPMDB altered outside of yum.
  Installing : libtool-ltdl-2.2.6-15.5.el6.x86_64                           1/4
  Installing : unixODBC-2.2.14-14.el6.x86_64                                2/4
  Installing : jpackage-utils-1.7.5-3.16.el6.noarch                         3/4
  Installing : 1:java-1.7.0-sun-1.7.0.25-xxb.el6.x86_64                     4/4
  Verifying  : 1:java-1.7.0-sun-1.7.0.25-xxb.el6.x86_64                     1/4
  Verifying  : unixODBC-2.2.14-14.el6.x86_64                                2/4
  Verifying  : jpackage-utils-1.7.5-3.16.el6.noarch                         3/4
  Verifying  : libtool-ltdl-2.2.6-15.5.el6.x86_64                           4/4

Installed:
  java-1.7.0-sun.x86_64 1:1.7.0.25-xxb.el6

Dependency Installed:
  jpackage-utils.noarch 0:1.7.5-3.16.el6  libtool-ltdl.x86_64 0:2.2.6-15.5.el6
  unixODBC.x86_64 0:2.2.14-14.el6

Complete!
 ---> 5cee7430f398
Removing intermediate container bf089eca1658
Step 6 : RUN yum install tomcat7-7.0.52 -y
 ---> Running in a847208ea2c5
Loaded plugins: fastestmirror
Setting up Install Process
Determining fastest mirrors
 * base: mirrors.zju.edu.cn
 * epel: mirrors.ustc.edu.cn
 * extras: mirrors.zju.edu.cn
 * updates: mirrors.zju.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package tomcat7.x86_64 1:7.0.52-xxb.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package          Arch            Version                    Repository    Size
================================================================================
Installing:
 tomcat7          x86_64          1:7.0.52-xxb.el6           xxb          6.4 M

Transaction Summary
================================================================================
Install       1 Package(s)

Total download size: 6.4 M
Installed size: 7.2 M
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : 1:tomcat7-7.0.52-xxb.el6.x86_64                              1/1
  Verifying  : 1:tomcat7-7.0.52-xxb.el6.x86_64                              1/1

Installed:
  tomcat7.x86_64 1:7.0.52-xxb.el6

Complete!
 ---> 0852c52a7856
Removing intermediate container a847208ea2c5
Successfully built 0852c52a7856
You have mail in /var/spool/mail/root
~~~

~~~bash
[root@hj-t docker]# docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
xxb/centos6.8-tomcat7           latest              0852c52a7856        33 minutes ago      687.7 MB
xxb/centos6.8-image             latest              04a72f961cee        59 minutes ago      399.2 MB
<none>                          <none>              c55929c79d98        About an hour ago   399.2 MB
centos6.8-base                  latest              fe95b256d279        About an hour ago   399.2 MB
trusty                          latest              8b40146fae0a        5 days ago          0 B
<none>                          <none>              38b969ffff81        5 days ago          172.2 MB
ubuntu                          12.04               93932704ad15        2 weeks ago         139.3 MB
centos                          latest              7322fbe74aa5        13 months ago       172.2 MB
dl.dockerpool.com:5000/mysql    latest              3c6d7e5c8c1b        21 months ago       235.6 MB
dl.dockerpool.com:5000/java     7u65-jdk            bd8bd16075a0        21 months ago       562.7 MB
dl.dockerpool.com:5000/centos   latest              87e5b6b3ccc1        22 months ago       224 MB
~~~


### DockerUI

~~~bash
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock hypriot/rpi-dockerui

~~~


### Shipyard

~~~
docker run -p 9000:9000 --rm -v /var/run/docker.sock:/var/run/docker.sock shipyard/deploy start
~~~

