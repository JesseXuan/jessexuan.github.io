---
layout: post
title:  "IPSec VPN测试配置记录"
date:   2019-01-16 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
因为ipsec包已经被加密，所以没有相应密匙信息，wireshark是无法解密的。
介于目前小站大部分情况需要先通过建立ipsec VPN后，再与核心网通信，为
此写下解包步骤记录方便后续的测试。

IPSec原理概述                                                    {#IPsec}
====================================
GRE（Generic Routing Encapsulation，通用路由封装）协议是对某些网络层协议（如IP 和IPX）
的数据报文进行封装，使这些被封装的数据报文能够在另一个网络层协议（如IP）中传输。
+ GRE Tunnel是一个虚拟的点对点的连接，在实际中可以看成仅支持点对点连接的虚拟接口
+ GRE封装过程
![encap]({{ '/styles/images/log_file/vpn/gre_encap.jpg' | prepend: site.baseurl  }})
经GRE模块处理后，原IP头部已经被封装在新IP头部和GRE头部之后;
新IP头部的长度为20字节,新IP数据包的IP头部的协议号为47;
GRE头部的长度为4～20字节（根据实际配置而定）;
+ GRE报文格式
![grepacket]({{ '/styles/images/log_file/vpn/gre_packet.jpg' | prepend: site.baseurl  }})
GRE头部结构参照RFC1701定义;
前4字节是必须出现的,第5～20字节将根据第1字节的相关bit位信息，可选出现;
13~15bit 版本：需为0;
16~31bit 协议类型：常用的协议，例如IP协议为0800


IPSEC                                                    {#Ipsec}
====================================
+ 打开wireshark，选中ESP包
网络拓扑
![ipsec]({{ '/styles/images/log_file/vpn/ipsec.jpg' | prepend: site.baseurl  }})

IPSec与NAT                                                    {#Nat}
------------------------------------

由于GRE没有所谓传输层的端口，所以在网络链路上存在多个GRE连接就会出现不能进行NAPT转换。
因此某种程度上来说，GRE与NAT无法共存。若用户一定要使用GRE VPN，且中间链路存在NAT时，
一般通过VPN嵌套的方式来实现，譬如在GRE的隧道之上嵌套一层IPsec VPN隧道，通过IPsec可以
穿越NAT的特性来达到用户需求。

参考链接                                                    {#Ref}
====================================
IPSEC原理[https://www.cnblogs.com/xiaohuamao/p/8021850.html](https://www.cnblogs.com/xiaohuamao/p/8021850.html)

IPSec及IKE原理[https://www.jianshu.com/p/66b0c193db46](https://www.jianshu.com/p/66b0c193db46)

IPsec总结[http://blog.51cto.com/13596342/2164126](http://blog.51cto.com/13596342/2164126)

IPSec原理与配置[http://blog.51cto.com/yangshufan/2103655](http://blog.51cto.com/yangshufan/2103655)
