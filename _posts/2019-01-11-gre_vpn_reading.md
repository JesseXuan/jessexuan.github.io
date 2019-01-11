---
layout: post
title:  "GRE VPN解读记录"
date:   2019-01-11 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
介于路由终端VPN功能测试。

GRE通用路由封装                                                    {#Gre}
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

GRE L3                                                    {#Grel3}
------------------------------------

网络拓扑
![grel3]({{ '/styles/images/log_file/vpn/gre_l3.jpg' | prepend: site.baseurl  }})

```bash
#常用GRE L3 TUNNEL在Linux下创建命令
# ip tunnel add tunnel0 mode gre remote 10.56.88.66 local 192.168.22.196 ttl 245
# ip link set tunnel0 up mtu 1414
# ip addr add 172.16.168.1/30 dev tunnel0
# ip addr add 172.16.168.1/30 peer 172.16.168.2/30 dev tunnel0
# ip route add 192.168.150.0/24 dev tunnel0

#删除tunnel
# ip link set tunnel0 down
# ip tunnel del tunnel0
```

GRE L2                                                    {#Grel2}
------------------------------------

网络拓扑
![grel2]({{ '/styles/images/log_file/vpn/gre_l2.jpg' | prepend: site.baseurl  }})

```bash
#GRE L2桥接创建方式
# ip link add gretap1 type gretap remote 10.56.88.66 local 192.168.22.196 ttl 255 nopmtudisc
# ip link set dev gretap1 up
# brctl addbr br0
# brctl addif br0 gretap1
# brctl addif br0 eth0
```

GRE与NAT                                                    {#GreNat}
------------------------------------

由于GRE没有所谓传输层的端口，所以在网络链路上存在多个GRE连接就会出现不能进行NAPT转换。
因此某种程度上来说，GRE与NAT无法共存。若用户一定要使用GRE VPN，且中间链路存在NAT时，
一般通过VPN嵌套的方式来实现，譬如在GRE的隧道之上嵌套一层IPsec VPN隧道，通过IPsec可以
穿越NAT的特性来达到用户需求。

