---
layout: post
title:  "L2TP VPN搭建测试记录(二)"
date:   2019-01-18 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
抽空使用CentOS搭建了VPN环境，独立于先前的RouterOS环境。

L2TP原理可以浏览博文《L2TP VPN搭建测试记录(一)》。

网络环境概述                                                    {#Topo}
====================================
+ L2TP服务器系统CentOS7，xl2tpd软件
+ VPN服务器双网卡：wan--em1(192.168.22.198), lan--p1p1(192.168.110.60)
+ 业务服务器Server: 192.168.110.70
+ VPN客户端CPE：wan(10.56.88.66), lan(192.168.150.1)
+ 测试PC1：192.168.150.100
+ 测试Windows VPN客户端PC2：192.168.22.98

网络拓扑

![topo]({{ '/styles/images/log_file/vpn/l2tp/topo.jpg' | prepend: site.baseurl  }})

L2TP搭建步骤                                                    {#Steps}
====================================
+ 检查系统版本
+ 系统网卡地址信息
+ 加载PPP拨号模块
+ 安装PPP拨号软件
+ 添加EPEL源
+ 安装VPN软件xl2tpd
```bash
# yum install -y xl2tpd
```
![xl2tpd]({{ '/styles/images/log_file/vpn/l2tp/xl2tpd.png' | prepend: site.baseurl  }})

+ 配置/etc/xl2tpd/xl2tpd.conf文件
```bash
#/etc/xl2tpd/xl2tpd.conf为xl2tpd服务程序读取的主文件
#先备份原配置文件
#然后在主文件中去掉注释并修改监听IP
#修改VPN分配给虚拟接口的IP地址
#local ip是VPN服务器隧道虚拟接口IP
#ip range是分配给vpn客户端隧道虚拟接口的IP地址池
#服务器名称可以自行决定是否修改，关系不大
# cp xl2tpd.conf xl2tpd.conf_ori_bak
# vim xl2tpd.conf
```
![xl2tpdconf]({{ '/styles/images/log_file/vpn/l2tp/xl2tpdconf.png' | prepend: site.baseurl  }})

+ 配置/etc/ppp/options.xl2tpd关联文件
```bash
#去掉注释并修改下发给客户端的DNS服务器IP
#去掉注释mschap加密
# vim /etc/ppp/options.xl2tpd
```
![opxl2tpd]({{ '/styles/images/log_file/vpn/l2tp/opxl2tpd.png' | prepend: site.baseurl  }})

+ 设置VPN认证账号密码
```bash
#编辑添加VPN连接的用户名密码
#用户名	服务器 密码 客户端地址，*表示任意IP
# vim /etc/ppp/chap-secrets
```
![chap]({{ '/styles/images/log_file/vpn/l2tp/chap.png' | prepend: site.baseurl  }})

+ 修改内核转发参数
```bash
#VPN收到包解码后需要做出转发到相应IP上去
#因此需要开启内核转发
#添加net.ipv4.ip_forward=1到内核参数文件/etc/sysctl.conf
#读取内核配置文件立即生效
# vim /etc/sysctl.conf
# sysctl -p
```

+ 启用l2tpd
```bash
#关闭防火墙
# systemctl stop firewalld.service
# systemctl disable firewalld.service
# setenforce 0
# vim /etc/sysconfig/selinux
# 
# systemctl restart xl2tpd.service
#
#设置开机自启动pptpd
# systemctl enable xl2tpd.service
```

L2TP连接                                                    {#Link}
====================================

L2TP控制连接建立过程
![l2tpCont]({{ '/styles/images/log_file/vpn/l2tp/l2tp_cont.png' | prepend: site.baseurl  }})

L2TP数据连接报文
![l2tpData]({{ '/styles/images/log_file/vpn/l2tp/l2tp_data.png' | prepend: site.baseurl  }})

IPSec加密                                                    {#Ipsec}
====================================
按上述步骤搭建好之后，以linux(openwrt)的路由器充当VPN客户端若建立不成功，可注释options.xl2tpd里的crtscts,lock再尝试。

但是win7系统建立的L2TP必须通过ipsec加密才能建立隧道连接。

因此继续在VPN服务器上搭建ipsec框架。

+ 安装ipsec组件（openswan或者libreswan）
```bash
# yum install -y openswan
```
+ 配置修改ipsec主文件/etc/ipsec.conf
```bash
# cp ipsec.conf ipsec.conf_ori_bak
# vim ipsec.conf
```
![ipsecConf]({{ '/styles/images/log_file/vpn/l2tp/ipsec_conf.png' | prepend: site.baseurl  }})

+ 设置预共享密钥文件/etc/ipsec.secrets
![ipsecSec]({{ '/styles/images/log_file/vpn/l2tp/ipsec_psk.png' | prepend: site.baseurl  }})

+ 修改xl2tpd.conf关联ipsec
![ipsecXl2tpd]({{ '/styles/images/log_file/vpn/l2tp/ipsec_xl2tpd.png' | prepend: site.baseurl  }})

+ 修改内核转发功能/etc/sysctl.conf(将服务器内部的ip地址关系进行转发和映射)
```bash
# vim /etc/sysctl.conf
# sysctl -p
```
![ipsecSysctl]({{ '/styles/images/log_file/vpn/l2tp/ipsec_sysctl.png' | prepend: site.baseurl  }})

+ 检验测试ipsec的安装配置
```bash
# ipsec verify
#
# 检测报错后的解决方法,强制修改目前网络驱动状态
# echo  0 >/proc/sys/net/ipv4/conf/em1/rp_filter
# echo  0 >/proc/sys/net/ipv4/conf/em2/rp_filter
# .
# .
# .
# 再次执行ipsec verify确认没有错误警告
```
![ipsecVerify]({{ '/styles/images/log_file/vpn/l2tp/ipsec_verify.png' | prepend: site.baseurl  }})

+ 启用ipsec，xl2tpd
```bash
# systemctl start ipsec
# systemctl start xl2tpd.service
# 
# systemctl enable ipsec
```
+ 验证VPN建立情况【win7上直接建立l2tp/ipsec测试成功】
![ipsecWin7]({{ '/styles/images/log_file/vpn/l2tp/ipsec_win7.png' | prepend: site.baseurl  }})


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
Centos7.5 系统使用L2TP/IPSec搭建服务器[http://blog.51cto.com/5001660/2296490](http://blog.51cto.com/5001660/2296490)

l2tp ipsec centos7[http://blog.51cto.com/wsxxsl/1914678](http://blog.51cto.com/wsxxsl/1914678)
