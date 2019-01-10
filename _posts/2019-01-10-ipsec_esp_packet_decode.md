---
layout: post
title:  "ipsec ESP wireshark解包"
date:   2019-01-10 17:31:01 +0800
categories: Documents
tag: 教程
---

* content
{:toc}


前言
====================================
因为ipsec包已经被加密，所以没有相应密匙信息，wireshark是无法解密的。
介于目前小站大部分情况需要先通过建立ipsec VPN后，再与核心网通信，为
此写下解包步骤记录方便后续的测试。

测试步骤                                                    {#Steps}
====================================
+ 打开wireshark，选中ESP包
+ 右键选中ESP包，在“Ptotocol Preference”栏内勾选ESP相关的选项
![step1]({{ '/styles/images/log_file/esp_step1.jpg' | prepend: site.baseurl  }})
+ 选择ESP SAs...后添加解密信息，注意发送/接收两个方向都需要添加
![step2]({{ '/styles/images/log_file/esp_step2.jpg' | prepend: site.baseurl  }})
+ 添加信息完毕，点击Apply即可，返回即可呈现解密后的ESP包内容
![step3]({{ '/styles/images/log_file/esp_step3.jpg' | prepend: site.baseurl  }})

解密信息获取                                                    {#Info}
------------------------------------

在小站建立ipsec VPN后，输入以下命令可以获取到加解密信息。

```bash
# ip -s xfrm state
```
或者
```bash
# ip x s
```
