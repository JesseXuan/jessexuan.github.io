---
layout: post
title:  "PPTP VPN测试配置记录"
date:   2019-01-11 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
PPTP VPN搭建

PPTP/L2TP服务器搭建教程网上较多，可以参考Linux系统搭建相关博客文章；

本人实在太懒了，懒得去按网友文章的方式去安装搭建，毕竟环境不一样坑较多；

而我直接选择了用RouterOS这个软路由镜像，简单易用，搭建快，功能可视化。


PPTP                                                    {#PPTP}
====================================
+ 打开wireshark，选中ESP包
+ PPTP报文格式

![pptpPacket]({{ '/styles/images/log_file/vpn/pptp_packet.jpg' | prepend: site.baseurl  }})

网络拓扑

![pptpTop]({{ '/styles/images/log_file/vpn/pptp_top.jpg' | prepend: site.baseurl  }})

PPTP VPN隧道MTU问题                                                    {#GreMtu}
------------------------------------

由于创建VPN添加了封装报头，所以IP内部payload理应减小，否则容易出现payload过大造成丢包。

CentOS 加载GRE后，默认隧道接口的MTU为1476；所以建议Layer3模式设置隧道接口MTU<=1476；

Layer2模式下，增加了内层MAC信息，所以隧道接口MTU<=1462。

PPTP与NAT                                                    {#GreNat}
====================================
由于GRE没有所谓传输层的端口，所以在网络链路上存在多个GRE连接就会出现不能进行NAPT转换。
因此某种程度上来说，GRE与NAT无法共存。若用户一定要使用GRE VPN，且中间链路存在NAT时，
一般通过VPN嵌套的方式来实现，譬如在GRE的隧道之上嵌套一层IPsec VPN隧道，通过IPsec可以
穿越NAT的特性来达到用户需求。
