---
layout: post
title:  "CentOS7使用systemctl添加自定义服务"
date:   2019-03-15 11:31:01 +0800
categories: Documents
tag: Linux
---

* content
{:toc}


CentOS7 system服务				{#System}
------------------------
自CentOS7起，已经不再使用CentOS5/6的chkconfig命令管理开机自启动服务和自定义脚本服务了，而是使用管理unit的方式来控制开机自启动服务和添加自定义脚本服务。在在/usr/lib/systemd/system目录下包含了各种unit文件，有service后缀的服务unit，有target后缀的开机级别unit等。服务类别又分为服务又分为系统服务（system）和用户服务（user）。系统服务：开机不登陆就能运行的程序（常用于开机自启）。用户服务：需要登陆以后才能运行的程序。

服务.service配置文件                              {#Service}
------------------------
service配置文件分三部分：
+ [Unit]部分：设置管理启动顺序与依赖关系

1. Description=服务描述 （给出该服务的简单描述）
2. Documentation=路径或URL （给出该服务的文档位置）
3. After=服务.target或者服务.service （定义该服务在其它什么服务开启之后才启动）
4. Requires=服务.servie （强依赖关系，必须某服务启动，该服务才能正常启动）

+ [Service]部分：设置该服务进程行为

1. Type=simple/forking等 （默认是simple，及ExecStart字段启动的进程为主进程；而forking时表示ExecStart字段将以fork（）方式启动）
2. ExecStart=命令 （定义启动进程时执行的命令）
3. ExecReload=命令 （定义重启服务时执行的命令）
4. ExecStop=命令 （定义停止服务时执行的命令）

+ [Install]部分：设置怎样做到开机启动

1. WantedBy=multi-user.target/graphical.target (该设置非常重要，该字段表示服务所在Target服务组)

添加自定义脚本service                              {#Customize}
------------------------
1. 编写好自己的脚本，最好把脚本放在/usr/bin目录下，如/usr/bin/test.sh

2. 编写自定义脚本服务的配置文件test.service
```bash
# vim test.service
#
[Unit]
Description="这是一个test demo服务"
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=simple
ExecStart=/bin/bash /usr/bin/test.sh start
ExecStop=/bin/bash /usr/bin/test.sh stop

[Install]
WantedBy=multi-user.target
```

3. 拷贝test.service文件到/usr/lib/systemd/system目录下
```bash
# cp test.service /usr/lib/systemd/system/test.service
#
# systemctl start test.service     #启动test.service
#
# systemctl enable test.service    #设置开机启动test.service，disable关闭开机启动
#
# systemctl stop test.service      #停止test.service
```

参考                              {#Ref}
------------------------
CentOS7添加自定义脚本服务[https://www.cnblogs.com/wutao666/p/9781567.html](https://www.cnblogs.com/wutao666/p/9781567.html)
