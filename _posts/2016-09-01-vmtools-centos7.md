---
layout: post
title: Centos7模板的vmtools问题
description: ""
modified: 2016-09-01
tags: [centos, vmtools, vmware]
image:
  feature: vmtools.jpg
  
---

## Centos7模板的vmtools问题

Centos7由于自身问题，不能直接使用vmtools安装包

### 安装open-vm-tools

~~~bash
yum install open-vm-tools
~~~

从`http://packages.vmware.com/tools/keys`下载`VMware Packaging Public Keys`

### import pub key

~~~bash
rpm -import /tmp/VMWARE-PACKAGING-GPG-DSA-KEY.pub
rpm -import /tmp/VMWARE-PACKAGING-GPG-RSA-KEY.pub
~~~
 
### 编辑 /etc/yum.repos.d/vmware-tools.repo
 
~~~bash
[vmware-tools]
name = VMware Tools
baseurl = http://packages.vmware.com/packages/rhel7/x86_64/
enabled = 1
gpgcheck = 1
~~~

### 安装open-vm-tools-deploypkg

~~~bash
yum install open-vm-tools-deploypkg
systemctl restart vmtoolsd
~~~

###修改系统版本信息

~~~bash
rm -f /etc/redhat-release && touch /etc/redhat-release && echo "Red Hat Enterprise Linux Server release 7.0 (Maipo)" > /etc/redhat-release
~~~