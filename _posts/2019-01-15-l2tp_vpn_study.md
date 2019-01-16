---
layout: post
title:  "L2TP VPN测试配置记录"
date:   2019-01-15 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
L2TP VPN搭建

PPTP/L2TP服务器搭建教程网上较多，可以参考Linux系统搭建相关博客文章；

本人实在太懒了，懒得去按网友文章的方式去安装搭建，毕竟环境不一样坑较多；

而我直接选择了用RouterOS这个软路由镜像，简单易用，搭建快，功能可视化。

L2TP原理概述
====================================
L2TP(Layer2 Tunneling Protocol)

具体原理和解释可以参考文末的几篇博客文章。

L2TP                                                    {#L2TP}
====================================
+ L2TP包括两种消息类型：控制消息和数据消息
1. 控制消息
> 用于L2TP隧道和会话连接的建立，维护和拆除。在控制消息的传输过程中，使用消息丢失重传和定时检测隧道连通性等集中来保证控制消息传输的可靠性，支持对控制消息的流量控制和拥塞控制。
2. 数据消息
> 用于封装PPP数据帧并在隧道上传输。数据消息是不可靠传输，不重传丢失的数据报文，不支持流量控制和拥塞控制。

网络拓扑

![l2tpTop]({{ '/styles/images/log_file/vpn/l2tp_top.png' | prepend: site.baseurl  }})

![l2tpConf]({{ '/styles/images/log_file/vpn/l2tp_conf.png' | prepend: site.baseurl  }})

L2TP实测报文呈现(控制连接建立与真实业务数据连接报文，报文与链路中间节点抓取)

![l2tpCont]({{ '/styles/images/log_file/vpn/l2tp_cont_pk.png' | prepend: site.baseurl  }})

![l2tpData]({{ '/styles/images/log_file/vpn/l2tp_data_pk.png' | prepend: site.baseurl  }})

由PC1(192.168.150.10) ping PC2(192.168.22.100)在路由终端出口后被转换成L2TP服务器分配的

虚拟IP(10.0.0.10)。

BCP                                                    {#BCP}
====================================
桥接控制协议（BCP）主要负责配置点对点链接终端的桥接协议参数。BCP 与链路控制协议使用相同的包交换机制。当 PPP 还未 到达网络层协议阶段时，不能交换 BCP 数据包，且在到达该阶段之前收到的 BCP 数据包则被丢弃。
BCP协议通过PPP协议将两个远端的以太网数据链路打通，BCP建立后独立于PPP隧道，将不与任何PPP的IP地址接口有关系。

BCP连接测试
------------------------------------

网络拓扑

![bcpTop]({{ '/styles/images/log_file/vpn/l2tp_bcp_top.png' | prepend: site.baseurl  }})

![bcpConf]({{ '/styles/images/log_file/vpn/l2tp_bcp_conf.png' | prepend: site.baseurl  }})

BCP实测报文呈现(控制连接建立与真实业务数据连接报文，报文与链路中间节点抓取)

![bcpCont]({{ '/styles/images/log_file/vpn/l2tp_bcp_cont_pk.png' | prepend: site.baseurl  }})

![bcpData]({{ '/styles/images/log_file/vpn/l2tp_bcp_data_pk.png' | prepend: site.baseurl  }})

BCP创建成功后，路由终端下的PC可从远端的DHCP服务器直接拿到地址。

L2TP VPN隧道MTU问题                                                    {#Mtu}
====================================

L2TP MTU/MRU = 1500 - 38(outside IP header 20B, UDP header 8B,L2TP header 6B, PPP header 4B) = 1462B

所以正常的以太网环境里l2tp虚拟接口MTU设置要<=1462字节，如果是BCP承载则<=1446字节(bcp header 2B,inside ether 14B)；

如中间链路有其它环境节点承载，比如中间有LTE网络，需要重新评估计算，即再增加了LTE数据面GTP隧道（20+8+8）。

L2TP与NAT                                                    {#Nat}
====================================
待续...

L2TP与PPTP                                                    {#Comp}
====================================
+ PPTP要求互联网络为IP网络,L2TP只要求隧道媒介提供面向数据包的点对点连接,L2TP可以在IP(使用UDP),FR永久虚拟电路,ATM虚拟电路网络上使用
+ PPTP只能在两端点之间建立单一隧道，L2TP支持在两端点见使用多隧道。使用L2TP，用户可以针对不同的服务质量创建不同的隧道
+ L2TP可以提供包头压缩。当压缩包头时，系统开销占用4个字节，而PPTP协议要用6个字节
+ PPTP依靠MPPE提供加密服务，而L2TP依靠IPSec提供加密服务


参考链接                                                    {#Ref}
====================================
L2TP协议原理[https://wenku.baidu.com/view/c88887e383d049649b6658fa.html](https://wenku.baidu.com/view/c88887e383d049649b6658fa.html)

L2TP详解[https://blog.csdn.net/u013485792/article/details/50838272](https://blog.csdn.net/u013485792/article/details/50838272)

BCP协议[http://www.360doc.com/content/17/0402/16/1394672_642333217.shtml#](http://www.360doc.com/content/17/0402/16/1394672_642333217.shtml#)
