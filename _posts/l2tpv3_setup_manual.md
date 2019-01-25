---
layout: post
title:  "L2TPv3 VPN搭建测试记录"
date:   2019-01-18 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
目前大多数L2TP环境都是v2版本，xl2tpd,openl2tpd都无法支持v3版本。
作为L2TP扩展的L2TPv3是一项无状态协议，它没有内在的信令或用于保持活动状态的(keep-alive)机制。最初在RFC 2661中定义的L2TP是设计用于为面向包的数据网上的多条第2层电路提供动态隧道的。它定义了一种标准隧道方式，这种方式使一条或多条第3层网络上的类似电路的连接看起来像是客户位置之间点到点或点到多点链路。基础L2TP协议由用于动态创建、维护和拆除L2TP会话的控制协议，以及在IP连接的节点之间多路复用与多路分离第2层数据流的数据封装构成。

网络环境概述                                                    {#Topo}
====================================
+ L2TPv3服务器系统CentOS7，目前没有开源软件，只能使用ip模块静态方式
+ VPN服务器双网卡：wan--em1(192.168.22.198), lan--p1p1(192.168.110.60)
+ 业务服务器Server: 192.168.110.70
+ VPN客户端CPE：wan(10.56.88.66), lan(192.168.150.1)
+ 测试PC1：192.168.150.100
+ 测试Windows VPN客户端PC2：192.168.22.98

网络拓扑

![topo]({{ '/styles/images/log_file/vpn/l2tpv3/topo.jpg' | prepend: site.baseurl  }})

L2TP搭建步骤                                                    {#Steps}
====================================
+ 检查系统版本
+ 系统网卡地址信息
+ 加载PPP拨号模块
+ 安装PPP拨号软件
+ 添加EPEL源
+ 安装VPN软件xl2tpd
```bash
# yum install -y xl2tpd
```
![xl2tpd]({{ '/styles/images/log_file/vpn/l2tpv3/xl2tpd.png' | prepend: site.baseurl  }})

+ 配置/etc/xl2tpd/xl2tpd.conf文件

参考链接                                                    {#Ref}
====================================
L2TPv3 概述[https://blog.csdn.net/bingyu9875/article/details/78853028](https://blog.csdn.net/bingyu9875/article/details/78853028)

ubuntu-l2tpv3配置介绍[http://manpages.ubuntu.com/manpages/trusty/man8/ip-l2tp.8.html](http://manpages.ubuntu.com/manpages/trusty/man8/ip-l2tp.8.html)
