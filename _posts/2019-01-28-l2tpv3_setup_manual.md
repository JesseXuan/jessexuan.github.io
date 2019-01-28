---
layout: post
title:  "L2TPv3 VPN搭建测试记录"
date:   2019-01-28 10:31:01 +0800
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
+ VPN服务器双网卡：wan--em1(192.168.22.196), lan--p1p1(192.168.110.70)
+ VPN服务器双网卡: wan--em1(192.168.22.198), lan--p1p1(192.168.0.186)
+ openwrt路由终端CPE：wan(10.56.88.66), lan(192.168.150.1)
+ 测试PC1：192.168.0.115 or 192.168.150.100
+ 测试PC2：192.168.110.30

网络拓扑

![topo]({{ '/styles/images/log_file/vpn/l2tpv3/topo.jpg' | prepend: site.baseurl  }})

L2TPv3静态无管理隧道配置搭建                                                    {#Steps}
====================================
+ 创建隧道
```bash
#tunnel_id与peer_tunnel_id必须唯一，且与对端相应设置id一致
#local/remote地址为双方的WAN口可相互通信的IP地址
#隧道encap封装方式：ip or udp
#隧道封装成udp模式后，需要指定两端收发UDP端口
#
# ip l2tp add tunnel tunnel_id 3000 peer_tunnel_id 4000 \
                  encap udp local 1.2.3.4 remote 5.6.7.8 \
                  udp_sport 5000 udp_dport 6000
#
#
server196# ip l2tp add tunnel tunnel_id 101 peer_tunnel_id 501 local 192.168.22.196 remote 192.168.22.198 encap udp udp_sport 5000 udp_dport 6000
#
#
server198# ip l2tp add tunnel tunnel_id 501 peer_tunnel_id 101 local 192.168.22.198 remote 192.168.22.196 encap udp udp_sport 6000 udp_dport 5000
```

+ 创建会话并加入到指定隧道内
```bash
#session_id与peer_session_id必须唯一，且与对端相应设置id一致
#会话可选项cookie两边要相同，layer2specific头部类型：none，udp
#
# ip l2tp add session tunnel_id 3000 session_id 1000 \
              peer_session_id 2000
#
#
server196# ip l2tp add session tunnel_id 101 session_id 1001 peer_session_id 5001
#
#
server198# ip l2tp add session tunnel_id 501 session_id 5001 peer_session_id 1001
```

+ 启用虚拟接口
```bash
# ip link set l2tpeth0 up mtu 1488
```
+ layer3模式(IP over tunnel)并添加静态业务路由
```bash
#
server196# ip addr add 10.42.1.1 peer 10.42.1.2 dev l2tpeth0
server196#
server196# ip route add 192.168.0.0/24 via 10.42.1.2 dev l2tpeth0
#
#
#
server198# ip addr add 10.42.1.2 peer 10.42.1.1 dev l2tpeth0
server198#
server198# ip route add 192.168.110.0/24 via 10.42.1.1 dev l2tpeth0
```
![add]({{ '/styles/images/log_file/vpn/l2tpv3/add.png' | prepend: site.baseurl  }})

+ layer2模式(Ethernet mac over tunnel)
```bash
#虚拟接口不配置IP地址，并且与lan接口加入到相同的网桥上
# ip link set l2tpeth0 up mtu 1446
# brctl addbr br0
# brctl addif br0 eth0
# brctl addif br0 l2tpeth0
# ip link set br0 up
```
+ 查看隧道与会话信息
```bash
# ip l2tp show tunnel [tunnel_id "ID"]
# ip l2tp show session [tunnel_id "ID"] [session_id "ID"]
```
![show]({{ '/styles/images/log_file/vpn/l2tpv3/show.png' | prepend: site.baseurl  }})

+ 删除会话与隧道
```bash
# ip l2tp del session  tunnel_id "ID" session_id "ID"
# ip l2tp del tunnel tunnel_id "ID"
```
![del]({{ '/styles/images/log_file/vpn/l2tpv3/del.png' | prepend: site.baseurl  }})

+ CentOS7 l2tpv3封装成IP模式
```bash
#CentOS7暂时无法封装成IP模式，待查找原因...
# ip l2tp add tunnel tunnel_id 101 peer_tunnel_id 501 local 192.168.22.196 remote 192.168.22.198 encap ip
RTNETLINK answers: Protocol not supported
#
#openwrt编译iproute2-full版本可支持encap ip模式
```
+ wireshark解l2tpv3数据包
> 暂无法找到如何正确解码的方法...待后续搜索查找怎么通过wireshark正确呈现封装里的内容。

L2TPv3选项                                                    {#Opt}
====================================
+ 默认封装为UDP模式，有利于NAT穿越
+ MTU与分片，开启IPv4碎片整理模块有利于接收端重组分片
```bash
# modprobe nf_degrag_ipv4
```
+ 由于linux l2tpv3没有用到hello报文检测链路状态，所以与其它l2tpv3软件对接的时候，这些软件需要关闭hello
+ 与不同l2tpv3软件对接的时候注意会话的 Layer2SpecificHeader type要一致

参考链接                                                    {#Ref}
====================================
L2TPv3 概述[https://blog.csdn.net/bingyu9875/article/details/78853028](https://blog.csdn.net/bingyu9875/article/details/78853028)

ubuntu-l2tpv3配置介绍[http://manpages.ubuntu.com/manpages/trusty/man8/ip-l2tp.8.html](http://manpages.ubuntu.com/manpages/trusty/man8/ip-l2tp.8.html)
