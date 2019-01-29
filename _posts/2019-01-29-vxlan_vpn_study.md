---
layout: post
title:  "Vxlan VPN搭建测试记录"
date:   2019-01-29 10:31:01 +0800
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
+ 环境简述
1. 服务器系统CentOS7，安装openvswitch
2. 服务器198双网卡：wan–em1(192.168.22.198), lan–p1p1
3. 服务器199双网卡：wan–em1(192.168.22.199), lan–p1p1
4. 测试PC1：192.168.0.x
5. 测试PC2：192.168.0.x
6. 可网管交换机SW1
+ 网络拓扑

![vxlan]({{ '/styles/images/log_file/vpn/vxlan.jpg' | prepend: site.baseurl  }})

+ 安装openvswitch
+ 创建配置vxlan
```bash
#openvswitch创建
# ovs-vsctl add-br br0;
# ovs-vsctl add-port br0 eth0
# ovs-vsctl add-port br0 vx0 -- set interface vx0 type=vxlan options:remote_ip=192.168.118.8 option:df_default=false
```

隧道封装/分片/MTU                                                    {#Mtu}
====================================


参考文章                                                    {#Ref}
====================================
技术发烧友：认识VXLAN[https://forum.huawei.com/enterprise/zh/thread-334207.html](https://forum.huawei.com/enterprise/zh/thread-334207.html)

VXLAN技术研究[https://blog.csdn.net/sinat_31828101/article/details/50504656](https://blog.csdn.net/sinat_31828101/article/details/50504656)

OpenvSwitch完全使用手册[https://blog.csdn.net/tantexian/article/details/46707175](https://blog.csdn.net/tantexian/article/details/46707175)

OVS常用操作[https://blog.csdn.net/qq_15437629/article/details/45951015](https://blog.csdn.net/qq_15437629/article/details/45951015)

基于centos7系统的Open vSwitch简单实验[https://blog.csdn.net/kid728/article/details/81209699](https://blog.csdn.net/kid728/article/details/81209699)

CentOS7上实践Open vSwitch+VXLAN[https://blog.csdn.net/dylloveyou/article/details/53365769](https://blog.csdn.net/dylloveyou/article/details/53365769)
