---
layout: post
title: "Debian的安装与基础配置"
date:   2024-03-02
tags: [R2S,Linux,Debian]
category: Linux
comments: true
author: Moorigin
---

## Debian的安装
以下内容基于 Debian12 版本，默认在root权限下操作。  
安装Debian的第一步从[官方网站](https://www.debian.org/releases/bookworm/debian-installer/)下载对应的ISO（本文使用 Debian12 操作），然后用工具烧录进U盘。  
具体安装步骤和细节可以查看：[寒山](https://www.nicvos.com/archives/10.html)、[CSDN](https://blog.csdn.net/weixin_44200186/article/details/131970040)

## 配置系统静态IP及DNS服务  
执行 `vim /etc/network/interfaces` ,添加以下内容:
```
auto eth1
iface eth1 inet static
address 192.168.2.10
netmask 255.255.255.0
gateway 192.168.2.1
dns-nameservers 192.168.2.1 223.5.5.5
```
> 前两行声明我们正在为 `eth1` 网络接口设置一个静态IP地址  
> `address` 填入想更改的IP  
> `netmask` 填入子网掩码（默认为`255.255.255.0`，不了解的同志使用默认的网络掩码）  
> `gateway` 填入网关地址  
> `dna-nameserver` 填入DNS服务器（建议设置为网关地址或者223.5.5.5）  

执行 `systemctl restart networking` 重启网络  

## 更换软件源
1、以清华源为例编辑 /etc/apt/sources.list 文件  
```
vim /etc/apt/sources.list
```
2、注释官方源并添加清华源  
```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

deb https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
# deb-src https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
```
3、更新软件列表  
```
apt update
```

## 安装Docker及Docker-Compose
虽然阿里ECS之类的云主机大多预装了docker，但是海外主机、自己装的linux系统大多还是没有docker的，我们从安装开始。以 `Debian` 为例。  
apt仓库提供的docker-io等软件包是社区构建的非官方版本，要安装官方版，需要多几个步骤。
### 使用官方安装脚本自动安装
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

### 手动安装
#### 1、Docker安装    

- 卸载旧版本（如果机器上没装过，则忽略）

```
apt remove -y docker docker-engine docker.io containerd runc
```

- 安装一些必备的软件包

```
apt update
apt upgrade -y
apt install curl vim wget gnupg dpkg apt-transport-https lsb-release ca-certificates
```

- 添加docker官方的gpg

```
curl -sSL https://download.docker.com/linux/debian/gpg | gpg --dearmor > /usr/share/keyrings/docker-ce.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-ce.gpg] https://download.docker.com/linux/debian $(lsb_release -sc) stable" > /etc/apt/sources.list.d/docker.list
```

- 国内机器可以用 `清华TUNA` 的国内源

```
curl -sS https://download.docker.com/linux/debian/gpg | gpg --dearmor > /usr/share/keyrings/docker-ce.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-ce.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian $(lsb_release -sc) stable" > /etc/apt/sources.list.d/docker.list
```

- 用新添加的docker软件包来进行升级更新，然后通过apt源安装docker

```
apt update
apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

此时可以使用 docker version 命令检查是否安装成功

#### 2、Docker-Compose安装  

因为我们已经安装了 docker-compose-plugin，所以 Docker 目前已经自带 docker compose 命令，基本上可以替代 docker-compose：  
```
root@debian ~ # docker compose version
Docker Compose version v2.18.1
```
如果某些镜像或命令不兼容，则我们还可以单独安装 Docker Compose：  
我们可以使用 Docker 官方发布的 Github 直接安装最新版本：  
```
curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```
此时可以使用 docker-compose version 命令检查是否安装成功：  
```
root@debian ~ # docker-compose version
Docker Compose version v2.18.1
```

> 更加具体的步骤和细节可以到官网查看：[docker安装](https://docs.docker.com/engine/install/debian/)、[docker-compose安装](https://docs.docker.com/compose/install/standalone/)。  

## 更换Docker源
Docker中国区官方镜像:  
https://registry.docker-cn.com  
ustc:  
https://docker.mirrors.ustc.edu.cn  
中国科技大学:  
https://docker.mirrors.ustc.edu.cn  
清华大学:  
https://docker.mirrors.tuna.tsinghua.edu.cn  
网易:  
http://hub-mirror.c.163.com  
阿里云:  
https://cr.console.aliyun.com  
腾讯云：  
https://mirror.ccs.tencentyun.com  

执行 `vim /etc/docker/daemon.json` 添加以下内容(选择自己喜欢的添加):  
```
{
    "registry-mirrors" : [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://cr.console.aliyun.com",
    "https://mirror.ccs.tencentyun.com",
    "https://docker.mirrors.tuna.tsinghua.edu.cn"
    ]
}
```
执行 `systemctl restart docker` 重启Docker  

## 优化TCP窗口,开启内核转发  
1、使用脚本  
```
wget --no-check-certificate -O tools.sh https://raw.githubusercontent.com/Moorigin/Linux/main/tools.sh && chmod +x tools.sh && ./tools.sh
```
下载速度或无法下载，通过能访问外网的设备去[项目地址](https://github.com/Moorigin/Linux)下载deb文件,然后通过sftp上传，然后执行bash tools.sh  
```
bash tools.sh
```
2、直接修改 `sysctl.conf` 文件  
执行 `vim /etc/sysctl.conf` ,添加以下内容:  
```
net.ipv4.ip_forward=1
```
最后执行 `sysctl -p /etc/sysctl.conf` 生效  

