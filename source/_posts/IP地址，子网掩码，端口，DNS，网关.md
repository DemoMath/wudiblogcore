---
layout: post
title:  "IP地址，子网掩码，端口，DNS，网关"
date:   2016-08-15
categories: [计算机网络,基础]
tags: [Net]
---
### IP地址
IP包头：

![](http://ot0nm27pk.bkt.clouddn.com/net_ip_01.png)

IP地址：

	00000000.00000000.00000000.00000000
	1 1 1 1 1 1 1 1 . 1 1 1 1 1 1 1 1 . 1 1 1 1 1 1 1 1 . 1 1 1 1 1 1 1 1

换算成为：

	0.0.0.0
	255.255.255.255

IP地址分类：

![](http://ot0nm27pk.bkt.clouddn.com/net_ip_02.png)


### 子网掩码

ip和子网掩码不能单独使用，必须一起使用（不能分开查看，要一起查看）

标准子网掩码：
	
	255.0.0.0  
	255.255.0.0  
	255.255.255.0

表示对应的数变化就是不同网段，需要路由器才能相互访问，其他的变化没关系，还是同一个网段

A类IP：

![](http://ot0nm27pk.bkt.clouddn.com/net_ziwang_01.png)

B类IP：
          
![](http://ot0nm27pk.bkt.clouddn.com/net_ziwang_02.png)

C类IP：

![](http://ot0nm27pk.bkt.clouddn.com/net_ziwang_03.png)
             
变长子网掩码及子网规划:

![](http://ot0nm27pk.bkt.clouddn.com/net_ziwang_04.png)



### 端口

在传输层确定端口号

通过ip能确定对方服务器位置，服务器上开启了一些服务，不同服务有不同的端口号，端口号就是为了确定目标服务器相应的服务

端口号

	2的16次方  0 - 65535

TCP协议包头：

![](http://ot0nm27pk.bkt.clouddn.com/net_baotou_01.png)

UDP协议包头：

![](http://ot0nm27pk.bkt.clouddn.com/net_baotou_02.png)
          
常见端口号：

	* FTP（文件传输协议）端口号：20 21
	* SSH（安全shell协议）：端口号 22
	* telnet（远程登录协议）：端口号 23（禁用，明文传输）
	* DNS（域名系统）：端口号53
	* http（超文本传输协议）：端口号80
	* SMTP（简单邮件传输协议）：端口号25
	* POP3（邮局协议3代）：端口号 110


查看本机启用的端口命令：

netstat -an

     -a：查看所有连接和监听端口
     -n：显示IP地址和端口号，而不显示域名和服务名

### DNS

不配置DNS是不能访问互联网的

Domain Name System的缩写，域名系统的缩写，又叫名称解析

名称解析概述

	* 在互联网中，通过IP地址来进行通信
	* IP地址用数字表示，记忆起来比较困难
	* 人对域名更加敏感


在DNS出现之前，我们有hosts文件（静态IP和域名对应），优先级比DNS更高

从Hosts文件到DNS

* 早期Hosts文件解析域名 
	1. 名称解析效能下降
	2. 主机维护困难

* DNS服务
	1. 层次性
	2. 分布式



DNS服务的作用

* 将域名解析为IP地址

		1. 客户机向DNS服务器发送域名查询请求
		2. DNS服务器告知客户机Web服务器的IP地址
		3. 客户机与Web服务器通信


域名空间结构

![](http://ot0nm27pk.bkt.clouddn.com/net_dns_01.png)


* 根域
* 顶级域（域名分配组织ISO分配）
	1. 组织域（gov政府，com商业，edu教育，org民间组织，net网络服务，mil军事）
	2. 国家或地区域(cn中国，hk香港，jp日本，uk英国，au澳大利亚)
* 二级域
* 主机名

DNS查询过程

![](http://ot0nm27pk.bkt.clouddn.com/net_dns_02.png)

DNS查询类型

* 从查询方式上区分
	* 递归查询：要么做出查询成功响应，要么作出查询失败的响应，一般客户机和服务器之间属于递归查询，即当客户机向DNS服务器发出请求后，若DNS服务器本身不能解析，则会向另外的DNS服务器发出查询请求，得到结果后转交给客户机
	* 迭代查询：服务器收到一次迭代查询回复一次结果，这个结果不一定是目标IP与域名的映射关系，也可以是其他DNS服务器的地址

* 从查询内容上区分
	* 正向查询由域名查找IP地址
	* 反向查询由IP地址查找域名

### 网关

概念：

* 网关（Gateway）又称网间连接器，协议转换器
* 网关在网络层以上实现网络互联，是最复杂的网络互连设备，仅用于两个高层协议不同的网络互联
* 网管既可以用于广域网互联，也可以用于局域网互联。
* 网关是一种充当转换重任的服务器或路由器

示意图：

![](http://ot0nm27pk.bkt.clouddn.com/net_wangguan_01.png)
