---
layout:     post
title:      "Ubuntu防火墙设置"
subtitle:   ""
date:       2018-11-10 10：57：42
author:     "Patrick"
header-img: ""
catalog:    true
tags:
    - Security
    - Linux
---


防火墙是服务器的安全保障，可以有效防止一些基本的网络安全威胁，比如DDoS攻击等。通过配置防火墙规则可以保证服务器应用和数据的安全，同时也可以避免一些服务器进行的端口扫描。

以下的命令是基于Ubuntu系统的操作命令，centos需要自行查阅其他资料。

服务器一般都默认安装了iptables，可以利用以下命令查看：


	whereis iptables


输出：


	iptables: /sbin/iptables /etc/iptables.rules /etc/iptables /usr/share/iptables /usr/share/man/man8/iptables.8.gz


如没有没有以上显示，则表示未安装，执行以下命令安装：


	apt-get install iptables


查看防火墙规则：


	iptables -L -n


输出：

	
	Chain INPUT (policy ACCEPT)
	target prot opt source destination
	Chain FORWARD (policy ACCEPT)
	target prot opt source destination
	Chain OUTPUT (policy ACCEPT)
	target prot opt source destination


添加规则：

	#允许本机
	iptables -A INPUT -i lo -j ACCEPT
	#双向
	iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
	
	iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
	iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
	iptables -I INPUT -s 202.38.254.0/24 -p tcp --dport 22 -j ACCEPT
	#如果限制ip了就不需要下面的各种策略设置。
	
	#丢弃坏的TCP包
	iptables -A FORWARD -p tcp ! --syn -m state --state NEW -j DROP
	#处理IP碎片数量,防止攻击,允许每秒100个
	iptables -A FORWARD -f -m limit --limit 100/s --limit-burst 100 -j ACCEPT
	
	#设置ICMP包过滤,允许每秒1个包,限制触发条件是10个包
	iptables -A FORWARD -p icmp -m limit --limit 1/s --limit-burst 10 -j ACCEPT
	
	#防止外部的ping和SYN洪水攻击
	iptables -A INPUT -p tcp --syn -m limit --limit 100/s --limit-burst 100 -j ACCEPT
	#ping洪水攻击，限制每秒的ping包不超过10个
	iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s –limit-burst 10 -j ACCEPT
	#防止各种端口扫描，将SYN及ACK SYN限制为每秒钟不超过200个
	iptables -A INPUT -p tcp -m tcp --tcp-flags SYN,RST,ACK SYN -m limit --limit 20/sec --limit-burst 200 -j ACCEPT
	#最后规则拒绝所有不符合以上所有的
	iptables -A INPUT -j DROP	

但是iptables是开机自动清除的，并不会持久化，所以如果想要对设置的策略进行保存，可以借助 iptables-persistent。
安装命令：

	sudo apt-get install iptables-persistent

利用以下命令保存设定的规则，就可以将设定的规则进行持久化，下次开机任然有效。

	iptables-save > /etc/iptables/rules.v4
	ip6tables-save > /etc/iptables/rules.v6