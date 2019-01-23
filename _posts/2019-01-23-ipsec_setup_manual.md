---
layout: post
title:  "IPSec VPN搭建测试记录(二)"
date:   2019-01-23 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
待续。。。

IPSec概述                                                    {#IPsec}
====================================
待续。。。

IPSEC搭建                                                    {#Ipsecbuild}
====================================
+ 测试环境描述
1. IPSEC服务器系统CentOS7，libreswan软件
2. VPN服务器198双网卡：wan--em1(192.168.22.198), lan--p1p1(192.168.0.186)
3. VPN服务器196双网卡：wan--em1(192.168.22.196), lan--p1p1(192.168.110.70)
4. 测试PC1：192.168.0.115, gw 192.168.0.186
5. 测试PC2：192.168.110.30, gw 192.168.110.70
+ 测试实验网络拓扑

![topo]({{ '/styles/images/log_file/vpn/ipsec/topo.png' | prepend: site.baseurl  }})

+ 安装ipsec套件
```bash
#在两个CentOS7服务器上安装ipsec软件libreswan
# yum install -y libreswan
```

+ 配置开启数据转发
```bash
# vim /etc/sysctl.conf
#在此文件末尾加上net.ipv4.ip_forward=1，net.ipv4.conf.default.rp_filter=0
#
#关闭icmp重定向
# sysctl -a | egrep "ipv4.*(accept|send)_redirects" | awk -F "=" '{print$1"= 0"}' >> /etc/sysctl.conf
#
# sysctl -p
```

+ 配置ipsec主文件/etc/ipsec.conf
```bash
#两个服务器分别修改相应IP和网段
#
#
```
![ikev1]({{ '/styles/images/log_file/vpn/ipsec/ipsec_conf_ikev1.png' | prepend: site.baseurl  }})

+ 配置ipsec密钥文件/etc/ipsec.secrets
> vim /etc/ipsec.secrets
```bash
#include /etc/ipsec.d/*.secrets
192.168.22.198  %any:   PSK "jesse"
```

+ 启动ipsec服务
```bash
#在两个服务器上执行启动ipsec服务进程
# systemctl start ipsec
#
#启动服务进程后验证ipsec
# ipsec verify
#
#libreswan监听在UDP的500和4500两个端口，其中500是用来IKE密钥交换协商，4500的NAT-T是nat穿透的
# netstat -anp | grep pluto
#
#接着手动创建连接
# ipsec auto --up site196-to-site198
#
#手动关闭ipsec connect连接ipsec auto --down site196-to-site198
#由于上述需要手动创建连接，可以将件/etc/ipsec.conf内的auto=add更改为auto=start则可启动ipsec时就可自动进行连接。
```
![upStatus1]({{ '/styles/images/log_file/vpn/ipsec/ipsec_up_status1.png' | prepend: site.baseurl  }})

测试结果                                                    {#Result}
====================================

IKE v1                                                    {#V1}
------------------------------------

根据上述配置测试结果可以看出流程为ike v1模式。

![pkcont]({{ '/styles/images/log_file/vpn/ipsec/ipsec_pk_cont.png' | prepend: site.baseurl  }})

解码后呈现(关于wireshark如何解码ESP包可参考先前博文《IPSec ESP wireshark解包》)

![pkdata]({{ '/styles/images/log_file/vpn/ipsec/ipsec_pk_data.png' | prepend: site.baseurl  }})

IKE v2                                                    {#V2}
------------------------------------

在/etc/ipsec.conf添加ikev2=insist强制启用ike v2模式。

![ikev2]({{ '/styles/images/log_file/vpn/ipsec/ipsec_conf_ikev2.png' | prepend: site.baseurl  }})

![upStatus2]({{ '/styles/images/log_file/vpn/ipsec/ipsec_up_status2.png' | prepend: site.baseurl  }})

参考链接                                                    {#Ref}
====================================
CentOS 6.3下基于Openswan IPSec VPN的实现[https://blog.csdn.net/bytxl/article/details/50457568](https://blog.csdn.net/bytxl/article/details/50457568)

保护虚拟私用网络（VPN）[https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/security_guide/sec-securing_virtual_private_networks](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/security_guide/sec-securing_virtual_private_networks)
