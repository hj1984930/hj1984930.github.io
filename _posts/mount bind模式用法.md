---
layout: post
title: mount bind模式用法
description: ""
modified: 2016-09-04
tags: [centos, mount, bind]
image:
  feature: linux.jpeg
  
---

### mount bind模式用法

功能：把某个目录 mount 到另外的目录
使用场景：
 
~~~bash
/dev/sdc1  /ftp
/dev/sdb1  /upload
~~~

此时想把/ftp下的upload目录映射到/upload目录
第一想法是使用软链接，ln -s /ftp/upload /upload，不幸的是ftp不支持软链接
此时，可以使用mount --bind的方式：

~~~bash
mount --bind /upload /ftp/upload
~~~

为了保证每次开机启动都自动挂载，写入/etc/fstab文件

~~~bash
/upload /ftp/upload none bind 0 0
~~~