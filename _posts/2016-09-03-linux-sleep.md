---
layout: post
title: linux开机延迟启动服务
description: ""
modified: 2016-09-03
tags: [centos, sleep,]
image:
  feature: disk.png
  
---

### linux开机延迟启动服务

在开机启动服务的时候，服务之间有依赖关系，必须在某个服务完全启动后才能启动其他的服务。在这种情况下，可以用到服务延迟启动的功能。
具体步骤如下：

1. 写一个实现延时启动的脚本myscript，在第一行加入延迟内容（myscript记得+x权限）

~~~
sleep 180   # 延时启动3分钟
......
~~~

2. 修改文件/etc/rc.local

~~~
nohup /path/to/myscript &
~~~

注意如果需要延时启动的程序已经做成自启动服务，需要取消掉

~~~
chkconfig  xxx off
~~~