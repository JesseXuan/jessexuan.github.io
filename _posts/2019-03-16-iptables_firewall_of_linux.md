---
layout: post
title:  "Iptables--Linux防火墙"
date:   2019-03-16 11:31:01 +0800
categories: Documents
tag: Linux
---

* content
{:toc}


Linux防火墙iptables				{#Iptables}
------------------------
netfilter/iptables（简称为iptables）组成Linux平台下的包过滤防火墙，与大多数的Linux软件一样。iptables是一个配置防火墙规则的工具，netfilter才是真正的核心，是Linux内核的安全模块。

四表五链                              {#Tables}
------------------------
+ 防火墙四表
1. filter表：负责过滤数据包功能，内核模块：iptables_filter
2. Nat表：网络地址转换功能，内核模块：iptable_nat
3. Mangle表：拆解修改数据包的服务类型、TTL、并且可以配置路由实现QOS等，重新封装数据包，内核模块：iptable_mangle
4. Raw表：决定数据包是否被状态跟踪机制处理，内核模块：iptable_raw

+ 核心预定义的五链
1. INPUT，发到本机的数据包应用此链中的策略列表
2. OUTPUT，本机发出去的数据包应用此链中的策略列表
3. FORWARD，通过本机转发的数据包应用此链中的策略列表
4. PREROUTING，进入网卡后，被路由前的数据包应用此链中的策略列表，所有数据包进来的时候都先由此链处理
5. POSTROUTING，被路由后，离开网卡前的数据包应用此链中的策略列表，所有数据包出去的时候都最后由此链处理

+ 规则表处理顺序

优先级次序由高到低：Raw —> mangle —> nat —> filter

+ 常用场景
1. 发给本机某进程的报文： PREROUTING --> INPUT
2. 由本机转发的报文：PREROUTING --> FORWARD --> POSTROUTING
3. 由本机某进程发出去的报文：OUTPUT --> POSTROUTING

规则Rules                              {#Rules}
------------------------
链内的策略规则是由上到下的匹配原则，所以要注意规则添加时最前面的优先级最高。

示例Examples                              {#Exam}
------------------------
1. 阻止往外网发包：
```bash
# iptables -t filter -I FORWARD -j DROP
# iptables -t filter -I OUTPUT -o wan -j DROP (记得指定wan网口，默认是使能于全部网口，会导致LAN断开的)
# iptables --flush (有的系统不能及时刷新时使用)
```

参考                              {#Ref}
------------------------
iptables详解[https://www.cnblogs.com/metoy/p/4320813.html](https://www.cnblogs.com/metoy/p/4320813.html)

iptables详解1[http://www.zsythink.net/archives/1199/](http://www.zsythink.net/archives/1199/)
