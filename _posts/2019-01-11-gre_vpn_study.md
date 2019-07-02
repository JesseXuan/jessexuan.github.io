---
layout: post
title:  "GRE VPN搭建测试记录"
date:   2019-01-11 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
介于路由终端VPN功能测试。

本篇属于VPN系列的第一篇，关于常用VPN协议的原理可以参考

[GRE、PPTP、L2TP隧道协议](https://blog.csdn.net/eydwyz/article/details/54879808)

GRE通用路由封装                                                    {#Gre}
====================================
GRE（Generic Routing Encapsulation，通用路由封装）协议是对某些网络层协议（如IP 和IPX）
的数据报文进行封装，使这些被封装的数据报文能够在另一个网络层协议（如IP）中传输。
+ GRE Tunnel是一个虚拟的点对点的连接，在实际中可以看成仅支持点对点连接的虚拟接口

+ GRE封装过程

![encap]({{ '/styles/images/log_file/vpn/gre_encap.png' | prepend: site.baseurl  }})

经GRE模块处理后，原IP头部已经被封装在新IP头部和GRE头部之后;

新IP头部的长度为20字节,新IP数据包的IP头部的协议号为47;

GRE头部的长度为4～20字节（根据实际配置而定）;

+ GRE报文格式

![grepacket]({{ '/styles/images/log_file/vpn/gre_packet.png' | prepend: site.baseurl  }})

GRE头部结构参照RFC1701定义;

前4字节是必须出现的,第5～20字节将根据第1字节的相关bit位信息，可选出现;

13~15bit 版本：需为0;

16~31bit 协议类型：常用的协议，例如IP协议为0800

GRE L3                                                    {#Grel3}
------------------------------------

该模式是目前最常用的IP in IP隧道方式，这样两端内网即可互访。

网络拓扑(R1:路由终端，R2:CentOS服务器)

没有支持GRE VPN的路由终端，可以用CentOS加载gre模块，雷同服务器端设置。

![grel3]({{ '/styles/images/log_file/vpn/gre_l3.png' | prepend: site.baseurl  }})

```bash
#常用GRE L3 TUNNEL在Linux下创建命令
# ip tunnel add tunnel0 mode gre remote 10.56.88.66 local 192.168.22.196 ttl 245
# ip link set tunnel0 up mtu 1414
# ip addr add 172.16.168.1/30 dev tunnel0
# ip addr add 172.16.168.1/30 peer 172.16.168.2/30 dev tunnel0

#删除tunnel
# ip link set tunnel0 down
# ip tunnel del tunnel0
```
路由终端上GRE Layer3配置

![grel3conf]({{ '/styles/images/log_file/vpn/gre_l3_conf.png' | prepend: site.baseurl  }})

GRE VPN建立的时候是不可靠的，类似UDP传输一样，不需要对方的确认
GRE隧道建立后的报文呈现：
+ 路由终端启用NAT时(源地址在WAN口出去时被替换成VPN tunnel local ip)

在PC1 192.168.150.10 ping VPN服务器LAN 192.168.110.70

192.168.150.10出了R1便被转换成172.16.168.2了

![grel3nat1]({{ '/styles/images/log_file/vpn/gre_l3_nat1.png' | prepend: site.baseurl  }})

![grel3nat2]({{ '/styles/images/log_file/vpn/gre_l3_nat2.png' | prepend: site.baseurl  }})

+ 路由终端禁用NAT时（没有出现源地址进行NAT转换，所以需要在目的端添加回程路由）

192.168.150.10直接透传出R1

![grel3router]({{ '/styles/images/log_file/vpn/gre_l3_no_nat.png' | prepend: site.baseurl  }})

目前PC1只能访问到R2的LAN口，无法访问到PC2，所以仍需要在R2做进一步设置
```bash
#在VPN服务器或者VPN终端上添加需要访问远端的流量路由进隧道内传输
# ip route add 192.168.150.0/24 dev tunnel0
#
#开启路由转发
# sysctl -w net.ipv4.ip_forward=1
#或者写进配置文件永久生效
#将文件/etc/sysctl.conf里面的net.ipv4.ip_forward=1的注释去除
# sysctl -p               --加载一下
```

另外还需要在远端局域网网关或者主机上添加静态路由，这样两边PC即可通过对方真实IP互访。
```bash
#如此例中，在192.168.110.1上添加静态路由
# route add -net 192.168.150.0/24 gw 192.168.110.70
```
设置完毕后，PC1上可以直接ping通PC2地址。

GRE L2                                                    {#Grel2}
------------------------------------

该模式可称之为ETH in GRE，以太网二层MAC可传输在隧道内，为两端建立一个统一的局域网。
网络拓扑

![grel2]({{ '/styles/images/log_file/vpn/gre_l2.png' | prepend: site.baseurl  }})

```bash
#GRE L2桥接创建方式(eth0为R2的LAN口)
# ip link add gretap1 type gretap remote 10.56.88.66 local 192.168.22.196 ttl 255 nopmtudisc
# ip link set dev gretap1 up
# brctl addbr br0
# brctl addif br0 gretap1
# brctl addif br0 eth0

#删除layer2
# brctl delif br0 gretap1
# brtcl delif br0 eth0
# ip link set br0 down
# brctl delbr br0
# ip link set  gretap1 down
# ip link del dev gretap1
```
路由终端上GRE Layer2配置

![grel2conf]({{ '/styles/images/log_file/vpn/gre_l2_conf.png' | prepend: site.baseurl  }})

两边配置完毕后，可将路由终端R1 LAN口的DHCP server服务关闭，让PC1从远端获取到地址。

PC1可以动态获取到PC2相同网段IP，可以正常互相通信。

![grel2payload]({{ '/styles/images/log_file/vpn/gre_l2_payload.png' | prepend: site.baseurl  }})

注意：两端报文格式flags标志需要保持一致，否则会导致互不认包Layer2不通的情况，如下key标志导致的：

![grel2source]({{ '/styles/images/log_file/vpn/gre_l2_right.png' | prepend: site.baseurl  }})

![grel2dest]({{ '/styles/images/log_file/vpn/gre_l2_error.png' | prepend: site.baseurl  }})

GRE Tunnel MTU问题                                                    {#MTU}
------------------------------------

由于创建VPN添加了封装报头，所以IP内部payload理应减小，否则容易出现payload过大造成丢包。

CentOS 加载GRE后，默认隧道接口的MTU为1476(1500-20ip-4gre)；所以建议Layer3模式设置隧道接口MTU<=1476；

Layer2模式下，增加了内层MAC信息，所以隧道接口MTU<=1462。


GRE与NAT                                                    {#GreNat}
====================================

由于GRE没有所谓传输层的端口，所以在网络链路上存在多个GRE连接就会出现不能进行NAPT转换。
因此某种程度上来说，GRE与NAT无法共存。若用户一定要使用GRE VPN，且中间链路存在NAT时，
一般通过VPN嵌套的方式来实现，譬如在GRE的隧道之上嵌套一层IPsec VPN隧道，通过IPsec可以
穿越NAT的特性来达到用户需求。

参考：[传统VPN与NAT穿越的兼容性](http://www.h3c.com/cn/d_201206/747032_97665_0.htm#)
