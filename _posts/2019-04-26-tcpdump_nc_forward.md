---
layout: post
title:  "tcpdump & nc 同用实时转发报文抓包"
date:   2019-04-26 11:31:01 +0800
categories: Documents
tag: Linux
---

* content
{:toc}


tcpdump&nc				{#Tcpdump}
------------------------
1. CentOS7下使用GUI的Wireshark老是core dumped异常关闭
2. 单纯在CentOS7下tcpdump抓包显现不够直观
3. nc命令可以监听或者转发数据
4. ADVsock2pipe监听接收转发的报文

```bash
# 在Linux上抓包并转发报文到指定的计算机上
# 启动抓包和转发报文前，需要在指定的计算机上开启监听程序
# 如在192.168.22.88上开启监听接收端口7777
# 在Linux上抓指定的p2p2接口的报文并且转发
# tcpdump -U -s 0 -i p2p2 not port 7777 -w - | nc 192.168.22.88 7777
#
# 在Linux上指定p2p2且指定某主机报文并且转发
# tcpdump -U -s 0 -i p2p2 host 8.8.8.8 and not port 7777 -w - | nc 192.168.22.88 7777
```

参考                              {#Ref}
------------------------
