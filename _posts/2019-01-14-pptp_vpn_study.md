---
layout: post
title:  "PPTP VPN测试配置记录"
date:   2019-01-14 10:31:01 +0800
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

PPTP原理概述
====================================
PPTP(Point to Point Tunneling Protocol)，即点对点隧道协议。
该协议是在PPP协议的基础上开发的一种新的增强型安全协议，
支持多协议虚拟专用网，可以通过密码验证协议，可扩展认证协议
等方法增强安全性。远程用户可以通过ISP、直接连接Internet或者
其他网络安全地访问企业网；

    它能够将PPP(点到点协议)帧封装成IP数据包，以便能够在基于
IP的互联网上进行传输。PPTP使用TCP是实现隧道的创建、维护与终
止，并使用GRE(通用路由封装)将PPP帧封装成隧道数据。被封装后
的PPP帧的有效载荷可以被加密或压缩；

    PPTP通信过程中需要建立两种连接，一种是控制连接，另一种是
数据连接。控制连接用来协商通信过程中的参数和进行数据连接的维
护。而真正的数据通信部分则交由PPTP数据连接完成。

    MPPE只提供连接加密，而不提供端-端加密。端-端加密属于应用
层的加密技术，如果应用中要求实现端-端加密，则可在PPTP隧道建
立之后，使用IPSec对两端的IP数据流进行加密处理。

    具体原理和解释可以参考文末的几篇博客文章。

PPTP                                                    {#PPTP}
====================================
+ PPTP连接控制报文与数据报文
+ PPTP控制连接建立步骤
1. 客户端与服务器TCP1723建立TCP连接
2. PPTP控制连接（control-connection-request/reply）和GRE隧道建立（outgoing-call-request/reply）
3. PPP协议的LCP协商
4. PPP协议的身份验证
5. PPP协议的NCP协商
6. PPP协议的CCP协商
+ PPTP数据连接的多层封装
1. 原始网络层数据报文encrypted PPP payload
2. 封装PPP header
3. 封装GRE header
4. 外层IP header

网络拓扑

![pptpTop]({{ '/styles/images/log_file/vpn/pptp_top.png' | prepend: site.baseurl  }})

![pptpConf]({{ '/styles/images/log_file/vpn/pptp_conf.png' | prepend: site.baseurl  }})

PPTP实测报文呈现(控制连接建立与真实业务数据连接报文，报文与链路中间节点抓取)

![pptpCont]({{ '/styles/images/log_file/vpn/pptp_cont_pk.png' | prepend: site.baseurl  }})

![pptpData]({{ '/styles/images/log_file/vpn/pptp_data_pk.png' | prepend: site.baseurl  }})

由PC1(192.168.150.10) ping PC2(192.168.22.100)在路由终端出口后被转换成PPTP服务器分配的

虚拟IP(172.31.0.10)。

PPTP VPN隧道MTU问题                                                    {#GreMtu}
------------------------------------

PPTP MTU/MRU = 1500 - 40(outside IP header 20B, GRE header 16B, PPP header 4B) = 1460B

所以正常的以太网环境里pptp虚拟接口MTU设置要<=1460字节，如中间环节还有穿插了别的环境节点

需要重新评估，比如中间有LTE网络，即再增加了LTE数据面GTP隧道（20+8+8）。

PPTP与NAT                                                    {#GreNat}
====================================
由于PPTP的数据连接使用的是GRE封装，而GRE天生与NAT存在冲突，多个GRE连接无法穿越NAT，
因GRE没有所谓传输层的端口，所以在网络链路上存在多个GRE连接就会出现不能进行NAPT转换。
所以NAT网关一般要求用ALG功能，PPTP ALG是通过GRE callid模拟端口进行转换。

参考链接                                                    {#Ref}
====================================
PPTP协议握手流程分析[https://blog.csdn.net/hdxlzh/article/details/46711901](https://blog.csdn.net/hdxlzh/article/details/46711901)

PPTP 理解以及报文的分析[https://blog.csdn.net/zhaqiwen/article/details/10083025](https://blog.csdn.net/zhaqiwen/article/details/10083025)

PPTP穿透NAT之深入分析[https://blog.csdn.net/eydwyz/article/details/54879787](https://blog.csdn.net/eydwyz/article/details/54879787)

Centos7.5 系统使用pptpd搭建服务器[http://blog.51cto.com/5001660/2177407](http://blog.51cto.com/5001660/2177407)
