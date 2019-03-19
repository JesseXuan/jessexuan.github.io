---
layout: post
title:  "CnetOS7最小安装后环境配置（laptop）"
date:   2019-03-19 11:31:01 +0800
categories: Documents
tag: Linux
---

* content
{:toc}


网络配置				{#NetConfig}
------------------------
1. CentOS7以最小安装模式完成系统安装，笔记本安装的时候尽量启用配置WiFi连接
2. 完成安装后，很多工具和软件没有安装的，需要手动安装
3. 用惯了ifconfig，但是最小安装没有带net-tools工具包，所以进行挂载系统盘安装
```bash
# 没有net-tools工具包情况下可以使用ip命令
# 如ip link show ,  ip addr show， ip route等
#
# 插上系统安装U盘或者光盘
# fdisk -l
# mount
# cd /mnt
# ls
# mkdir upan
# ls
# mount /dev/sdb1 /mnt/upan
# cd upan/Packages/
# ls | grep net-tools
# rpm -ivh net-tools-2.0-0.24.20131004git.el7.x86_64.rpm
# ifconfig
```

本地yum安装源                              {#Yum}
------------------------
为了方便没有网络情况下也可以安装常用的软件和工具，同时解决软件包的依赖关系，配置本地yum安装源十分必要。
```bash
# 下载拷贝ISO系统镜像包到本地进行挂载
# mount -t iso9660 -o,loop CentOS-7-x86_64-DVD-1810.iso /mnt/upan
# echo "/root/tmp_downloads/CentOS-7-x86_64-DVD-1810.iso /mnt/upan" >> /etc/fstab
#
# 为了每次开机自动挂载目录，可以配置挂载文件
# vim /etc/fstab

#
# /etc/fstab
# Created by anaconda on Sun Dec 23 10:04:34 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=035b3d07-4c7f-4cf0-a176-830ca35eca87 /boot                   xfs     defaults        0 0
/dev/mapper/centos-home /home                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
/root/tmp_downloads/CentOS-7-x86_64-DVD-1810.iso /mnt/upan	iso9660	loop 0 0

#
```

配置本地yum repo仓库
```bash
# cd /etc/yum.repos.d/
# ls
# cp CentOS-Base.repo CentOS-Base.repo_original.bak
# vim CentOS-Base.repo

# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
baseurl=file:///mnt/upan
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

# 配置Yum源后清理更新缓存
# yum clean all
# yum repolist
```

WiFi配置                                {#WlanConfig}
------------------------
系统起来后，发现wifi没有自动连接上网络，所以手动设置启用
```bash
# wifi的命令管理工具是iw，首先安装
# yum install -y iw
# ifconfig
# iw dev    列出系统无线设备
# iw wlp12s0 link    查看无线网卡链路状态
# ip link set wlp12s0 up    启用无线网卡
# iw wlp12s0 scan | grep SSID    扫描周边无线网络，列出SSID
# wpa_supplicant -B -i wlp12s0 -c <(wpa_passphrase "TP-LINK_xxx" "jesse@xuan")   连接wpa指定SSId的无线网络
# dhclient wlp12s0  无线网卡动态获取地址信息等
# ifconfig   查看是否获取到IP地址
# ping www.baidu.com   测试网络连通性
```

由于上述命令步骤都是即时生效，重启计算机后需要重新执行。所以可将这些命令步骤放在脚本内，以方便一键执行。
```bash
# vim /usr/bin/wlp12s0_up.sh

#!/bin/bash
#Description: This script for wifi auto connection

sleep 30
iw dev >>/dev/null 2>&1
iw wlp12s0 link >>/dev/null 2>&1
ip link set wlp12s0 up >>/dev/null 2>&1
wpa_supplicant -B -i wlp12s0 -c <(wpa_passphrase TP-LINK_xxx jesse@xuan)
dhclient wlp12s0 >>/dev/null 2>&1

#设置更新系统时间
sleep 20
ntpdate cn.pool.ntp.org >>/dev/null 2>&1

# 赋予执行权限
# chmod 755 /usr/bin/wlp12s0_up.sh
#
```

上述脚本终究需要每次开机后，执行一次脚本，所以把脚本自定义为系统服务来启动即可设置开机自动运行（脚本放在/etc/rc.local不生效）
```bash
# 先创建编辑一个自定义脚本服务文件
# vim wifi_connection.service

[Unit]
Description="This is my laptop wifi auto connection after poweron"
Requires=network.target
After=network.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/bin/wlp12s0_up.sh

[Install]
WantedBy=multi-user.target

# 把该配置文件拷贝到/usr/lib/systemd/system目录下，并且设置开机启动
# cp wifi_connection.service /usr/lib/systemd/system/wifi_connection.service
# systemctl enable test.service
```


Python3安装                              {#Python}
------------------------
CentOS7系统安装后默认自带python2.7，python3.x版本需要手动安装
```bash
# 本次安装最新的python3.7.1版本
# 先安装依赖的库和软件包等
# yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel xz-devel libpcap-devel libffi-devel
# yum -y install tk-devel gdbm-devel db4-devel
# yum -y groupinstall "Development tools"
#
# 下载python3.7.1源码包到本地
# wget https://www.python.org/ftp/python/3.7.1/Python-3.7.1.tar.xz
# ls
# tar -xvJf Python-3.7.1.tar.xz
# mkdir /usr/local/python3
# cd Python-3.7.1
# ./configure --prefix=/usr/local/python3
# make && make install
#
# 安装后设置python3快捷命令
# which python
# cd /bin
# ls -l python*
# ln -s /usr/local/python3/bin/python3 /bin/python3
# ln -s /usr/local/python3/bin/pip3 /bin/pip3
# python3
# yum install -y epel-release
# yum install -y python-pip
```

Docker安装                              {#Docker}
------------------------
Docker安装环境和docker基础镜像下载
```bash
# 先安装依赖包
# yum -y install yum-utils device-mapper-persistent-data lvm2
#
# 配置docker yum安装源并在线安装最新版本
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# yum list docker-ce --showduplicates | sort -r
# yum -y install docker-ce
#
# 启动docker服务
# systemctl start docker
# systemctl enable docker
#
# 运行测试docker/查看版本/查看本地镜像/搜索远程库的系统镜像/拉取下载指定系统镜像到本地
# docker run hello-world
# docker version
# docker images
# docker search centos
# docker pull centos
# docker search debian
# docker pull debian
# docker pull ubuntu
# docker images
```

参考                              {#Ref}
------------------------
centos7安装python3.7.1[https://www.cnblogs.com/felixwang2/p/9934460.html](https://www.cnblogs.com/felixwang2/p/9934460.html)

Centos7下安装Docker[https://www.cnblogs.com/qgc1995/p/9553572.html](https://www.cnblogs.com/qgc1995/p/9553572.html)
