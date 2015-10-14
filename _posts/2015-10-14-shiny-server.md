---
layout: post
title: Tomcat CPU100%故障排查
description: ""
modified: 2015-10-15
tags: [tomcat, java, CPU ]
image:
  feature: abstract-3.jpg

---

### Tomcat CPU100%故障排查
- - -

####现象描述

* 线上环境异常，无法正常访问
* 系统负载高cpu长时间超过90%，系统吞吐率低


####排查是否是安全攻击问题导致

####排查是否是数据库问题导致

####排查是否是云平台异常导致

####排查是否是tomcat本身异常导致

* 通过top命令查询系统负载情况
* 发现2173这个java进程负载较高，持续时间较长，出现故障
![top](/images/top.png)
* 进一步查看线程，确定是java进程出现异常
* 耗时较高的java线程，cpu占用情况很高
![thread](/images/thread.png)
* 进一步打印线程堆栈信息
![线程堆栈](/images/线程堆栈信息.png)

~~~java
[root@portal-app-2 logs]# jstack 2173 |grep db7 -A 30
"catalina-exec-282" daemon prio=10 tid=0x00007f838f7cf000 nid=0xdb7 waiting on condition [0x00007f83716d5000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
 - parking to wait for  <0x00000006150ac5c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
 at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
 at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
 at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
 at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)


"catalina-exec-274" daemon prio=10 tid=0x00007f839005f000 nid=0xdb6 waiting on condition [0x00007f8370dcc000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
 - parking to wait for  <0x00000006150ac5c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
 at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
 at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
 at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
 at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)


"catalina-exec-278" daemon prio=10 tid=0x00007f8398027000 nid=0xdb2 waiting on condition [0x00007f83710cf000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
[root@portal-app-2 logs]# jstack 2173 |grep db7 -A 60
"catalina-exec-282" daemon prio=10 tid=0x00007f838f7cf000 nid=0xdb7 waiting on condition [0x00007f83716d5000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
 - parking to wait for  <0x00000006150ac5c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
 at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
 at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
 at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
 at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)


"catalina-exec-274" daemon prio=10 tid=0x00007f839005f000 nid=0xdb6 waiting on condition [0x00007f8370dcc000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
 - parking to wait for  <0x00000006150ac5c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
 at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
 at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
 at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
 at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)


"catalina-exec-278" daemon prio=10 tid=0x00007f8398027000 nid=0xdb2 runnable [0x00007f83710cf000]
   java.lang.Thread.State: RUNNABLE
 at java.net.SocketInputStream.socketRead0(Native Method)
 at java.net.SocketInputStream.read(SocketInputStream.java:150)
 at java.net.SocketInputStream.read(SocketInputStream.java:121)
 at org.apache.coyote.http11.InternalInputBuffer.fill(InternalInputBuffer.java:516)
 at org.apache.coyote.http11.InternalInputBuffer.fill(InternalInputBuffer.java:501)
 at org.apache.coyote.http11.Http11Processor.setRequestLineReadTimeout(Http11Processor.java:173)
 at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:929)
 at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:589)
 at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.java:312)
 - locked <0x0000000799c676f8> (a org.apache.tomcat.util.net.SocketWrapper)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)


"catalina-exec-279" daemon prio=10 tid=0x00007f83840c6000 nid=0xdb1 waiting on condition [0x00007f8371bda000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
 - parking to wait for  <0x00000006150ac5c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
 at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
 at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
 at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
 at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)


"catalina-exec-277" daemon prio=10 tid=0x00007f83b4017000 nid=0xdb0 waiting on condition [0x00007f83711d0000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
[root@portal-app-2 logs]# jstack 2173 |grep db7 -A60
"catalina-exec-282" daemon prio=10 tid=0x00007f838f7cf000 nid=0xdb7 waiting on condition [0x00007f83716d5000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
 - parking to wait for  <0x00000006150ac5c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
 at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
 at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
 at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
 at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)


"catalina-exec-274" daemon prio=10 tid=0x00007f839005f000 nid=0xdb6 waiting on condition [0x00007f8370dcc000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
 - parking to wait for  <0x00000006150ac5c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
 at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
 at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
 at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
 at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)


"catalina-exec-278" daemon prio=10 tid=0x00007f8398027000 nid=0xdb2 runnable [0x00007f83710cf000]
   java.lang.Thread.State: RUNNABLE
 at java.net.SocketInputStream.socketRead0(Native Method)
 at java.net.SocketInputStream.read(SocketInputStream.java:150)
 at java.net.SocketInputStream.read(SocketInputStream.java:121)
 at org.apache.coyote.http11.InternalInputBuffer.fill(InternalInputBuffer.java:516)
 at org.apache.coyote.http11.InternalInputBuffer.fill(InternalInputBuffer.java:501)
 at org.apache.coyote.http11.Http11Processor.setRequestLineReadTimeout(Http11Processor.java:173)
 at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:929)
 at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:589)
 at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.java:312)
 - locked <0x0000000799c676f8> (a org.apache.tomcat.util.net.SocketWrapper)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)


"catalina-exec-279" daemon prio=10 tid=0x00007f83840c6000 nid=0xdb1 waiting on condition [0x00007f8371bda000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
 - parking to wait for  <0x00000006150ac5c0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
 at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
 at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
 at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
 at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
 at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
 at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
 at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
 at java.lang.Thread.run(Thread.java:724)


"catalina-exec-277" daemon prio=10 tid=0x00007f83b4017000 nid=0xdb0 waiting on condition [0x00007f83711d0000]
   java.lang.Thread.State: WAITING (parking)
 at sun.misc.Unsafe.park(Native Method)
~~~

* 线程信息转储

~~~
kill -3 3511
~~~

~~~java
"FD_SOCK client connection handler,LIFERAY-TRANSPORT-CHANNEL-0,portal-app-2-53253" daemon prio=10 tid=0x00007f8388088800 nid=0x4204 runnable [0x00007f835edec000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.read(SocketInputStream.java:150)
        at java.net.SocketInputStream.read(SocketInputStream.java:121)
        at java.net.SocketInputStream.read(SocketInputStream.java:203)
        at org.jgroups.protocols.FD_SOCK$ClientConnectionHandler.run(FD_SOCK.java:1116)
        at java.lang.Thread.run(Thread.java:724)


"liferay/scheduler_engine-4" daemon prio=10 tid=0x00007f83800a0800 nid=0x4202 waiting on condition [0x00007f84105f9000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000060aac2ba8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
        at com.liferay.portal.kernel.concurrent.TaskQueue.take(TaskQueue.java:254)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor._getTask(ThreadPoolExecutor.java:490)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor.access$8(ThreadPoolExecutor.java:469)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor$WorkerTask.run(ThreadPoolExecutor.java:598)
        at java.lang.Thread.run(Thread.java:724)


"jigsaw/permission/dataextuser/initDataExtUser-23" daemon prio=10 tid=0x00007f83980a6800 nid=0x40d8 waiting on condition [0x00007f83d905c000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000060a86a7d0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:226)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2082)
        at com.liferay.portal.kernel.concurrent.TaskQueue.poll(TaskQueue.java:194)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor._getTask(ThreadPoolExecutor.java:486)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor.access$8(ThreadPoolExecutor.java:469)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor$WorkerTask.run(ThreadPoolExecutor.java:598)
        at java.lang.Thread.run(Thread.java:724)


"jigsaw/permission/dataextuser/initDataExtUser-22" daemon prio=10 tid=0x00007f83bc041800 nid=0x3f27 waiting for monitor entry [0x00007f83d8f5b000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.ekingstar.jigsaw.permission.messaging.DataExtUserInitMessageListener.receive(DataExtUserInitMessageListener.java:36)
        - waiting to lock <0x000000060d0e8678> (a java.lang.Object)
        at com.liferay.portal.kernel.messaging.InvokerMessageListener.receive(InvokerMessageListener.java:72)
        at com.liferay.portal.kernel.messaging.ParallelDestination$1.run(ParallelDestination.java:69)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor$WorkerTask._runTask(ThreadPoolExecutor.java:682)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor$WorkerTask.run(ThreadPoolExecutor.java:593)
        at java.lang.Thread.run(Thread.java:724)


"jigsaw/permission/dataextuser/initDataExtUser-21" daemon prio=10 tid=0x00007f83a8409000 nid=0x3f26 runnable [0x00007f83d8b56000]
/nid
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000006142d3d88> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
        at com.liferay.portal.kernel.concurrent.TaskQueue.take(TaskQueue.java:254)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor._getTask(ThreadPoolExecutor.java:490)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor.access$8(ThreadPoolExecutor.java:469)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor$WorkerTask.run(ThreadPoolExecutor.java:598)
        at java.lang.Thread.run(Thread.java:724)


"jigsaw/permission/dataextuser/initDataExtUser-18" daemon prio=10 tid=0x00007f84144c7000 nid=0x7e8f waiting on condition [0x00007f8365695000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000060a86a7d0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:226)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2082)
        at com.liferay.portal.kernel.concurrent.TaskQueue.poll(TaskQueue.java:194)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor._getTask(ThreadPoolExecutor.java:486)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor.access$8(ThreadPoolExecutor.java:469)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor$WorkerTask.run(ThreadPoolExecutor.java:598)
        at java.lang.Thread.run(Thread.java:724)


"jigsaw/permission/dataextuser/initDataExtUser-17" daemon prio=10 tid=0x00007f8394079800 nid=0x7e8e waiting on condition [0x00007f8364e8d000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000060a86a7d0> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:226)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2082)
        at com.liferay.portal.kernel.concurrent.TaskQueue.poll(TaskQueue.java:194)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor._getTask(ThreadPoolExecutor.java:486)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor.access$8(ThreadPoolExecutor.java:469)
        at com.liferay.portal.kernel.concurrent.ThreadPoolExecutor$WorkerTask.run(ThreadPoolExecutor.java:598)
        at java.lang.Thread.run(Thread.java:724)


"FD_SOCK client connection handler,LIFERAY-TRANSPORT-CHANNEL-0,portal-app-2-53253" daemon prio=10 tid=0x00007f8388027800 nid=0x5622 runnable [0x00007f841aef7000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.read(SocketInputStream.java:150)
        at java.net.SocketInputStream.read(SocketInputStream.java:121)
        at java.net.SocketInputStream.read(SocketInputStream.java:203)
        at org.jgroups.protocols.FD_SOCK$ClientConnectionHandler.run(FD_SOCK.java:1116)
        at java.lang.Thread.run(Thread.java:724)


"FD_SOCK client connection handler,liferay-multi-vm-clustered,portal-app-2-23624" daemon prio=10 tid=0x00007f83a8091000 nid=0x5620 runnable [0x00007f835d8d7000]
~~~


####分析

* NEW：线程创建尚未启动。
* RUNNABLE：包括操作系统线程状态中的Ready和Running，可能在等待时间片或者正在执行。
* BLOCKED：线程被阻塞。
* WAITING：不会分配CPU执行时间，直到别的线程显示的唤醒，否者无限期等待。LockSupport.park()，没有设置Timeout参数的Object.wait()和Thread.join()，会导致此现象。
* TIMED_WAITING：不会分配CPU执行时间，直到系统自动唤醒，不需要别的线程显示唤醒。Thread.sleep()，LockSupport.parkNanos()，LockSupport.parkUntil()，设置了超时时间的Object.wait()和Thread.join()，会让线程进入有限期等待。
* TERMINATED：线程执行结束。

从上面的线程信息可以初步判定为死循环