---
layout: post
title: Shiny-Server安装配置
description: ""
modified: 2015-10-14
tags: [shiny-server, R ]
image:
  feature: abstract-2.jpg

---

###Shiny-Server安装配置
- - -

####R环境准备

* R环境已经有了
* 安装shiny包，install.packages('shiny')
* 安装rmarkdown包，install.packages('rmarkdown')
####下载rpm安装包

~~~
curl -L -O https://download3.rstudio.org/centos-6.3/x86_64/shiny-server-1.4.0.662-rh6-x86_64.rpm

yum install --nogpgcheck  shiny-server-1.4.0.662-rh6-x86_64.rpm
~~~

####配置修改
修改文件`/etc/shiny-server/shiny-server.conf`,端口改为8080

####应用程序
将shiny的自带例子复制到shiny-server下

~~~
cp -r /usr/lib64/R/library/shiny/examples/* /srv/shiny-server/
~~~

####重启服务

~~~
cd /opt/shiny-server/bin
stop shiny-server
start shiny-server
~~~

####访问
通过http://ip:8080/访问shiny-server服务
直接访问某个app：http://10.2.29.22:8080/01_hello

~~~
[root@client shiny-server]# ll /srv/shiny-server/
总用量 44
drwxr-xr-x 3 root root 4096 10月 13 15:59 01_hello
drwxr-xr-x 2 root root 4096 10月 13 15:59 02_text
drwxr-xr-x 2 root root 4096 10月 13 15:59 03_reactivity
drwxr-xr-x 2 root root 4096 10月 13 15:59 04_mpg
drwxr-xr-x 2 root root 4096 10月 13 15:59 05_sliders
drwxr-xr-x 2 root root 4096 10月 13 15:59 06_tabsets
drwxr-xr-x 2 root root 4096 10月 13 15:59 07_widgets
drwxr-xr-x 3 root root 4096 10月 13 15:59 08_html
drwxr-xr-x 2 root root 4096 10月 13 15:59 09_upload
drwxr-xr-x 2 root root 4096 10月 13 15:59 10_download
drwxr-xr-x 2 root root 4096 10月 13 15:59 11_timer
lrwxrwxrwx 1 root root   38 10月 13 14:59 index.html -> /opt/shiny-server/samples/welcome.html
lrwxrwxrwx 1 root root   37 10月 13 14:59 sample-apps -> /opt/shiny-server/samples/sample-apps
~~~ 

![shiny-server](/images/shiny-server.png)
![shiny-server-01-hello](/images/shiny-server-hello.png)