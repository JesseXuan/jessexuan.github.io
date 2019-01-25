---
layout: post
title:  "IPSec VPN搭建测试记录(一)"
date:   2019-01-22 10:31:01 +0800
categories: 数通
tag: VPN
---

* content
{:toc}


前言
====================================
通过先前的几种VPN实验可知，他们都是明文传输，存在安全风险。IPSec（IP Security）是IETF制定的为保证在Internet上传送数据的安全保密性的三层隧道加密协议。

IPSec原理概述                                                    {#IPsec}
====================================
IPSec不是指具体的一种协议，而是一套协议族。

+ IPSec提供的安全服务：
1. 数据机密性
2. 数据完整性
3. 数据来源认证
4. 防重放
+ IPSec的工作方式(工作方式不同，数据包封装的方式不一样)
1. 传输模式transport
2. 隧道模式tunnel
+ IPSec通信保护协议
1. AH 鉴别头部，协议号51，不能对用户数据加密，不支持NAT-T穿越
2. ESP 封装安全载荷，协议号50
+ 安全联盟SA
1. IPsec对数据流提供安全服务是通过安全联盟SA来实现的，SA包括了协议，算法，密钥等协商参数，关系到IP报文的处理

IPSEC与IKE                                                    {#IKE}
====================================
用IPsec保护一个IP包之前，必须先建立安全联盟（SA）。IPsec的安全联盟可以手工建立，但网络中节点较多时就出现问题了。

IKE因特网密钥交换协议是IPSec的信令协议，为IPSec提供了自动协商交换密钥，建立安全联盟的服务，简化IPSec的管理配置。

+ IKE的工作流程(共分为两个阶段)
1. 阶段一：先建立IKE SA，为阶段二提供保护和快速协商
2. 阶段二：在IKE SA的保护模式下继续为最终用户数据完成IPSec SA协商

+ IPSec与IKE的关系
1. IKE是UDP应用层协议，是IPSec的信令协议
2. IKE为IPSec协商建立安全联盟，并把建立的参数和生成的密钥交给IPSec
3. IPSec使用IKE建立好的SA对IP报文进行加密封装和验证处理
4. IKE阶段二协商出来的IPSec SA是单向的，为保护数据流而创建的SA
5. IPSec处理是基于IP层部分

参考链接                                                    {#Ref}
====================================
IPSEC原理[https://www.cnblogs.com/xiaohuamao/p/8021850.html](https://www.cnblogs.com/xiaohuamao/p/8021850.html)

IPSec及IKE原理[https://www.jianshu.com/p/66b0c193db46](https://www.jianshu.com/p/66b0c193db46)

IPsec总结[http://blog.51cto.com/13596342/2164126](http://blog.51cto.com/13596342/2164126)

IPSec原理与配置[http://blog.51cto.com/yangshufan/2103655](http://blog.51cto.com/yangshufan/2103655)

IPSEC实现分析[https://wenku.baidu.com/view/b776b9bcfe4733687f21aa59.html?pn=50](https://wenku.baidu.com/view/b776b9bcfe4733687f21aa59.html?pn=50)

IKEv1和IKEv2有哪些区别[https://wenku.baidu.com/view/1f52ba1d0a4e767f5acfa1c7aa00b52acfc79c7a.html](https://wenku.baidu.com/view/1f52ba1d0a4e767f5acfa1c7aa00b52acfc79c7a.html)
