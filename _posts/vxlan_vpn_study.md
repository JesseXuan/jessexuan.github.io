---
layout: post
title:  "Vxlan VPN搭建测试记录"
date:   2019-01-25 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
普通的VLAN数量只有4096个，无法满足大规模云计算IDC的需求。
xlan(virtual Extensible LAN)虚拟可扩展局域网，是一种overlay的网络技术，使用MAC in UDP的方法进
行封装，共50字节的封装报文头。

Vxlan                                                    {#Vxlan}
====================================
+ openvswitch软件

网络拓扑

![vxlan]({{ '/styles/images/log_file/vpn/vxlan.jpg' | prepend: site.baseurl  }})

```bash
#openvswitch创建
# ovs-vsctl add-br br0;
# ovs-vsctl add-port br0 eth0
# ovs-vsctl add-port br0 vx0 -- set interface vx0 type=vxlan options:remote_ip=192.168.118.8 option:df_default=false
```

