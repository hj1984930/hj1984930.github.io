---
layout: post
title: GPDB segment失效问题处理
description: ""
modified: 2015-09-24
tags: [Greenplum, 大数据, segment失效, ]
image:
  feature: abstract-4.jpg

---

###GPDB Segment失效问题处理

####1. 现象
重新加载配置时，报错

~~~
[gpadmin@mdw gpseg-1]$ gpstop -u
20150923:13:53:57:004760 gpstop:mdw:gpadmin-[INFO]:-Starting gpstop with args: -u
20150923:13:53:57:004760 gpstop:mdw:gpadmin-[INFO]:-Gathering information and validating the environment...
20150923:13:53:57:004760 gpstop:mdw:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20150923:13:53:57:004760 gpstop:mdw:gpadmin-[INFO]:-Obtaining Segment details from master...
20150923:13:53:59:004760 gpstop:mdw:gpadmin-[INFO]:-Greenplum Version: 'postgres (Greenplum Database) 4.3.3.0 build 1'
20150923:13:53:59:004760 gpstop:mdw:gpadmin-[INFO]:-Signalling all postmaster processes to reload
.
20150923:13:54:01:004760 gpstop:mdw:gpadmin-[CRITICAL]:-Error occurred: Error Executing Command:
 Command was: 'ssh -o 'StrictHostKeyChecking no' sdw1 ". /usr/local/greenplum-db/./greenplum_path.sh; $GPHOME/bin/pg_ctl reload -D /data1/primary/gpseg1"'
rc=1, stdout='', stderr='pg_ctl: PID file "/data1/primary/gpseg1/postmaster.pid" does not exist
Is server running?
'
~~~

从日志可以看出segment1有问题，没有起来

####2. 重启整个GP数据库，问题依旧

####3. 怀疑是segment故障，恢复segment

~~~
gprecoverseg
~~~
等几分钟后，segmentrecover完成

####5. 需重启DB

~~~
gpstop -M immediate
gpstart
~~~

####6. 查看系统状态，恢复正常
~~~
[gpadmin@mdw ~]$ gpstate -e
20150923:16:10:56:008479 gpstate:mdw:gpadmin-[INFO]:-Starting gpstate with args: -e
20150923:16:10:56:008479 gpstate:mdw:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 4.3.3.0 build 1'
20150923:16:10:56:008479 gpstate:mdw:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.2.15 (Greenplum Database 4.3.3.0 build 1) on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.4.2 compiled on Sep 23 2014 15:44:20'
20150923:16:10:56:008479 gpstate:mdw:gpadmin-[INFO]:-Obtaining Segment details from master...
20150923:16:10:57:008479 gpstate:mdw:gpadmin-[INFO]:-Gathering data from segments...
...
20150923:16:11:00:008479 gpstate:mdw:gpadmin-[INFO]:-----------------------------------------------------
20150923:16:11:00:008479 gpstate:mdw:gpadmin-[INFO]:-Segment Mirroring Status Report
20150923:16:11:00:008479 gpstate:mdw:gpadmin-[INFO]:-----------------------------------------------------
20150923:16:11:00:008479 gpstate:mdw:gpadmin-[INFO]:-All segments are running normally
~~~


####7. 查询某个节点是否有坏盘

~~~
[root@sdw2 ~]# omreport storage pdisk controller=0 -fmt tbl | awk -F"|" '{print $1"|"$2"|"$3"|"$4"|"$5"|"$9"|"$15"|"$18"|"$19}' |egrep -v '\-\-\-\-|\|\|'
ID    | Status  | Power Status  | Name                | State  | Secured       | Used RAID Disk Space             | Vendor ID| Product ID
0:0:0 | Ok      | Spun Up       | Physical Disk 0:0:0 | Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:1 | Ok      | Spun Up       | Physical Disk 0:0:1 | Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:2 | Ok      | Spun Up       | Physical Disk 0:0:2 | Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:3 | Ok      | Spun Up       | Physical Disk 0:0:3 | Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:4 | Ok      | Spun Up       | Physical Disk 0:0:4 | Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:5 | Ok      | Spun Up       | Physical Disk 0:0:5 | Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:6 | Ok      | Spun Up       | Physical Disk 0:0:6 | Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:7 | Ok      | Spun Up       | Physical Disk 0:0:7 | Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:8 | Critical| Not Applicable| Physical Disk 0:0:8 | Removed| Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:9 | Ok      | Spun Up       | Physical Disk 0:0:9 | Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:10| Ok      | Spun Up       | Physical Disk 0:0:10| Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
0:0:11| Ok      | Spun Up       | Physical Disk 0:0:11| Online | Not Applicable| 1,862.50 GB (1999844147200 bytes)| DELL     | WDC WD2003FYYS-18W0B0
~~~
可以看到节点2的第八块盘故障
