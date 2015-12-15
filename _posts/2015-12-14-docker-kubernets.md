---
layout: post
title: Centos7上Docker和kubernets环境部署
description: ""
modified: 2015-12-08
tags: [docker,kubernetes]
image:
  feature: abstract-8.jpg
  
---

###Centos7上Docker和kubernets环境部署

- - -

* Kubernetes 是 Google 团队发起的开源项目，它的目标是管理跨多个主机的容器，提供基本的部署，维护以及运用伸缩，主要实现语言为Go语言。Kubernetes是：

易学：轻量级，简单，容易理解
便携：支持公有云，私有云，混合云，以及多种云平台
可拓展：模块化，可插拔，支持钩子，可任意组合
自修复：自动重调度，自动重启，自动复制

* 

注意：关闭firewall和selinux

####安装etcd
* etcd是CoreOS团队发起的一个管理配置信息和服务发现（service discovery）的项目。它的目标是构建一个高可用的分布式键值（key-value）数据库，基于 Go 语言实现。

~~~
yum install etcd -y
~~~

编辑 `/etc/etcd/etcd.conf`

~~~
# [member]
ETCD_NAME=default
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"

#[cluster]
ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:2379"
~~~

启动服务

~~~
systemctl enable etcd
systemctl start etcd
~~~

####安装kubernets master

~~~
yum install kubernetes-master -y
~~~

编辑`/etc/kubernetes/apiserver`

~~~
# The address on the local server to listen to.
KUBE_API_ADDRESS="--address=0.0.0.0"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd_servers=http://10.2.29.185:2379"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

# default admission control policies
KUBE_ADMISSION_CONTROL="--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"

# Add your own!
KUBE_API_ARGS=""
~~~

编辑`/etc/kubernetes/controller-manager`

~~~
###
# The following values are used to configure the kubernetes controller-manager

# defaults from config and apiserver should be adequate

# Add your own!
KUBE_CONTROLLER_MANAGER_ARGS="--node-monitor-grace-period=10s --pod-eviction-timeout=10s"

~~~

编辑`/etc/kubernetes/config`

~~~
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow_privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://10.2.29.185:8080"
~~~

启动服务

~~~
systemctl enable kube-apiserver kube-scheduler kube-controller-manager
systemctl start kube-apiserver kube-scheduler kube-controller-manager
~~~

####安装kubernets node

~~~
yum install kubernetes-node flannel docker -y
~~~

编辑`/etc/kubernetes/config`

~~~
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow_privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://10.2.29.185:8080"
~~~

编辑`/etc/kubernetes/kubelet`

~~~

# kubernetes kubelet (minion) config

# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=127.0.0.1"

# The port for the info server to serve on
# KUBELET_PORT="--port=10250"

# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname_override=10.2.29.185"

# location of the api-server
KUBELET_API_SERVER="--api_servers=http://10.2.29.185:8080"

# Add your own!
KUBELET_ARGS="--pod-infra-container-image=kubernetes/pause"
~~~

启动服务

~~~
systemctl enable kubelet kube-proxy
systemctl start kubelet kube-proxy
~~~

####kubernets node配置flannel

* 初始化flannel的etcd配置

~~~
etcdctl -C 10.2.29.185:2379 set /coreos.com/network/config '{ "Network": "10.1.0.0/16" }'
~~~

编辑`/etc/sysconfig/flanneld`

~~~
# etcd url location.  Point this to the server where etcd runs
FLANNEL_ETCD="http://10.2.29.185:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
FLANNEL_ETCD_KEY="/coreos.com/network"

# Any additional options that you want to pass
#FLANNEL_OPTIONS=""
~~~

启动服务

~~~
systemctl enable flanneld
systemctl restart flanneld docker
~~~

查看docker进程

~~~
ps -ef|grep docker
root      3287     1  3 20:22 ?        00:00:00 /usr/bin/docker daemon --selinux-enabled --bip=10.1.51.1/24 --mtu=1472
~~~

网络已通

~~~
ping 10.1.51.1
PING 10.1.51.1 (10.1.51.1) 56(84) bytes of data.
64 bytes from 10.1.51.1: icmp_seq=1 ttl=64 time=0.088 ms
64 bytes from 10.1.51.1: icmp_seq=2 ttl=64 time=0.078 ms
64 bytes from 10.1.51.1: icmp_seq=3 ttl=64 time=0.072 ms
64 bytes from 10.1.51.1: icmp_seq=4 ttl=64 time=0.092 ms
^C
--- 10.1.51.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3000ms
rtt min/avg/max/mdev = 0.072/0.082/0.092/0.012 ms
~~~

登陆到docker中

~~~
[root@docker ~]# ssh 10.1.51.1
The authenticity of host '10.1.51.1 (10.1.51.1)' can't be established.
ECDSA key fingerprint is 27:f8:7f:92:88:43:ac:e7:be:08:45:88:70:b2:09:1d.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.1.51.1' (ECDSA) to the list of known hosts.
root@10.1.51.1's password:
Last login: Mon Dec 14 20:20:50 2015 from 10.2.29.162
[root@docker ~]#
[root@docker ~]#
[root@docker ~]#
[root@docker ~]# ifconfig
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.1.51.1  netmask 255.255.255.0  broadcast 0.0.0.0
        ether 02:42:ae:a2:21:1a  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.2.29.185  netmask 255.255.255.0  broadcast 10.2.29.255
        inet6 fe80::250:56ff:fea7:79ac  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:a7:79:ac  txqueuelen 1000  (Ethernet)
        RX packets 859  bytes 77851 (76.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 321  bytes 39175 (38.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

flannel0: flags=81<UP,POINTOPOINT,RUNNING>  mtu 1472
        inet 10.1.51.0  netmask 255.255.0.0  destination 10.1.51.0
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 8567  bytes 1089443 (1.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8567  bytes 1089443 (1.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
~~~

OK，至此结束