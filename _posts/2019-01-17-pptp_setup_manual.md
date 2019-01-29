---
layout: post
title:  "PPTP VPN搭建测试记录(二)"
date:   2019-01-17 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
抽空使用CentOS搭建了VPN环境，独立于先前的RouterOS环境。

PPTP原理可以浏览博文《PPTP VPN搭建测试记录(一)》。

网络环境概述                                                    {#Topo}
====================================
+ PPTP服务器系统CentOS7，pptpd软件
+ VPN服务器双网卡：wan--em1(192.168.22.198), lan--p1p1(192.168.110.60)
+ 业务服务器Server: 192.168.110.70
+ VPN客户端CPE：wan(10.56.88.66), lan(192.168.150.1)
+ 测试PC1：192.168.150.100
+ 测试Windows VPN客户端PC2：192.168.22.98

网络拓扑

![topo]({{ '/styles/images/log_file/vpn/pptp/topo.jpg' | prepend: site.baseurl  }})

PPTP搭建步骤                                                    {#Steps}
====================================
+ 检查系统版本
```bash
# cat /etc/redhat-release
```
![os]({{ '/styles/images/log_file/vpn/pptp/step1.png' | prepend: site.baseurl  }})

+ VPN服务器网卡地址信息
```bash
# ifconfig
```
![net]({{ '/styles/images/log_file/vpn/pptp/step2.png' | prepend: site.baseurl  }})

+ 加载PPP拨号模块
```bash
# modprobe ppp-compress-18 && echo "yes"
```
![pppcom]({{ '/styles/images/log_file/vpn/pptp/step3.png' | prepend: site.baseurl  }})

+ 安装PPP拨号软件
```bash
# yum install -y ppp
```
![ppp]({{ '/styles/images/log_file/vpn/pptp/step4.png' | prepend: site.baseurl  }})

+ 添加EPEL源
```bash
# wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
# rpm -ivh epel-release-latest-7.noarch.rpm
# yum repolist
```
![epel]({{ '/styles/images/log_file/vpn/pptp/step5.png' | prepend: site.baseurl  }})

+ 安装VPN软件PPTPD
```bash
# yum install -y pptpd
```
![pptpd]({{ '/styles/images/log_file/vpn/pptp/step6.png' | prepend: site.baseurl  }})

+ 配置/etc/pptpd.conf文件
```bash
#/etc/pptpd.conf为pptpd服务程序读取的主文件
#主要就是修改VPN分配给虚拟接口的IP地址
#localip是VPN服务器隧道虚拟接口IP
#remoteip是分配给vpn客户端隧道虚拟接口的IP地址池
# vim pptpd.conf
```
![pptpdconf]({{ '/styles/images/log_file/vpn/pptp/step7.png' | prepend: site.baseurl  }})

+ 配置/etc/ppp/options.pptpd关联文件
```bash
#去掉注释并修改下发给客户端的DNS服务器IP
#添加pptpd日志记录路径
# vim /etc/ppp/options.pptpd
```

1. 去掉注释并且修改相应的DNS IP，建议WAN口的DNS

![oppptp]({{ '/styles/images/log_file/vpn/pptp/step8_1.png' | prepend: site.baseurl  }})

2. 添加pptpd运行记录路径

![oppptp]({{ '/styles/images/log_file/vpn/pptp/step8_2.png' | prepend: site.baseurl  }})

+ 设置VPN认证账号密码
```bash
#编辑添加VPN连接的用户名密码
#用户名	服务器 密码 客户端地址，*表示任意IP
# vim /etc/ppp/chap-secrets
```
![chap]({{ '/styles/images/log_file/vpn/pptp/step9.png' | prepend: site.baseurl  }})

+ 修改内核转发参数
```bash
#VPN收到包解码后需要做出转发到相应IP上去
#因此需要开启内核转发
#添加net.ipv4.ip_forward=1到内核参数文件/etc/sysctl.conf
#读取内核配置文件立即生效
# vim /etc/sysctl.conf
# sysctl -p
```
![sysctl]({{ '/styles/images/log_file/vpn/pptp/step10.png' | prepend: site.baseurl  }})

+ 启用pptpd
```bash
#关闭防火墙
# systemctl stop firewalld.service
# systemctl disable firewalld.service
# setenforce 0
# vim /etc/sysconfig/selinux
# 
# systemctl restart pptpd.service
#
#设置开机自启动pptpd
# systemctl enable pptpd.service
```

PPTP连接                                                    {#Link}
====================================

PPTP VPN连接建立                                             {#Setup}
------------------------------------

PPTP控制连接建立过程
![pptpCont]({{ '/styles/images/log_file/vpn/pptp/pptp_cont.png' | prepend: site.baseurl  }})

PPTP数据连接报文
![pptpData]({{ '/styles/images/log_file/vpn/pptp/pptp_data.png' | prepend: site.baseurl  }})

MPPE加密                                                   {#Mppe}
------------------------------------

安装后PPTP默认启用了MPPE加密，可以看到wireshark无法显示PPP compressed Datagram的。
VPN两端都必须同时启用，否则CCP协商不通过导致隧道连接建立失败。

MPPE加密的PPTP数据报文
![mppe]({{ '/styles/images/log_file/vpn/pptp/mppe_pk.png' | prepend: site.baseurl  }})

pptpd服务器端的mppe配置【在上面step8里/etc/ppp/options.pptpd启用require-mppe-128】
![mppeconf1]({{ '/styles/images/log_file/vpn/pptp/step8_3.png' | prepend: site.baseurl  }})

路由终端（VPN客户端）的mppe配置
![mppeconf2]({{ '/styles/images/log_file/vpn/pptp/mppe_conf2.png' | prepend: site.baseurl  }})

Win7上创建的pptp连接默认是"需要加密"即mppe配置，如服务器端没有启用mppe，则选择“可选加密”，否则连接报“错误 628”

![win7pptp]({{ '/styles/images/log_file/vpn/pptp/win7_pptp.png' | prepend: site.baseurl  }})


通过网关VPN服务器连外网                                                    {#Gate}
====================================
如果需要通过VPN服务器连接外网，则需要在VPN服务器WAN口上做NAT转发；

设置转发规则，从源地址发出的所有包进行伪装，改变地址。
```bash
# iptables -t nat -A POSTROUTING -s 10.0.10.0/24 -o em1 -j MASQUERADE
# chmod +x /etc/rc.d/rc.local
# echo "iptables -t nat -A POSTROUTING -s 10.0.10.0/24 -o em1 -j MASQUERADE" >> /etc/rc.d/rc.local
```

参考链接                                                    {#Ref}
====================================
Centos7.5 系统使用pptpd搭建服务器[http://blog.51cto.com/5001660/2177407](http://blog.51cto.com/5001660/2177407)
