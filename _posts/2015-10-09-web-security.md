---
layout: post
title: 信息安全与web安全
description: ""
modified: 2015-10-09
tags: [信息安全, web安全 ]
image:
  feature: abstract-11.jpg

---

###信息安全与web安全
- - -


####整体网络安全架构
![网络安全架构](/images/网络架构.jpg)

* 网络层：思科ASA5500系列防火墙，传统的网络包检测和过滤
* 操作系统层：趋势Deep Security深度安全防护（防病毒、防恶意程序、防篡改、防火墙、漏洞发现与虚拟补丁），对2-7层进行防护，发现安全攻击行为实时阻断
* 威胁发现：趋势TDA威胁发现（发现所有的扫描行为、攻击行为及尝试攻击行为），对2-7层实时检测，发现攻击行为及时报警
* 应用层：F5ASMweb应用防火墙，对3-7层进行安全防护，发现应用攻击行为实时阻断
* 数据库审计：服务器负载均衡、websever、中间件、数据库四层关联审计

思科ASA5500系列防火墙：

* 主要工作在TCP/IP层以下
* 深度状态包检测过滤

F5 ASM应用层安全：

* 传统防火墙只能防止网络攻击，SSL封装的Web攻击对防火墙来说是不可见的
* 网络入侵检测是设计工作在TCP/IP层的，在HTTP层并没有效果
* 对内容检查的深度不够，且无法对内容进行任何操作
* 数据伪装的问题


####下一步的工作
* 渗透测试，kali
* 编码工具：小葵
* 渗透工具：菜刀
* 抓包工具：Burp Suite
* sql注入工具：sql注入模块、Pangolin、穿山甲、明小子、萝卜头Havij
* cookies修改和盗取cookie：啊D
* post注入和cookie注入工具：sqlmap
* 漏洞分析检测工具：webcruise
* 跨站神器：beef，xsser

![渗透测试](/images/渗透测试.bmp)