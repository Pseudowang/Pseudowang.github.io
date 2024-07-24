+++
title = 'Docker搭建OpenWrt'
date = 2024-06-23T21:27:28+08:00
draft =  false
categories = ["Tech"]
tags = ["Notes", "Basic", "Docker", "OpenWrt"]
+++

### 前言

这周五的时候，因为上周实训完了，学到了很多东西，学到了Gitlab，之前就有想自己搭建一个自己的代码版本管理仓库，正好宿舍有一台小服务器在Docker 中运行着OpenWRT， 一开始一直拉不下来 GitLab 的镜像，好像是因为最近很多国内docker镜像站给关闭了，然后后面通过加速站拉取下拉了，然后发现磁盘空间不够了， 然后自己手贱去调整了分区，然后就再也开不了机了(就是自己对这个方面不了解，然后去乱搞才会导致这件事情的发生)， 不过上次搭建OpenWRT 和 nas-tools 的时候没有写笔记，然后已经忘了差不多了。这次花了很多时间就是因为上次没有写笔记进行总结，才会花费了这么长的时间，所以这次打算把全部过程记录下来，这样给以后自己手贱搞坏服务器的时候可以用(最好还是别发生，真的很累)
### Docker 安装

这个可以直接搜索，因为现在使用的官方安装脚本已经不能用了，不知道到时候再用的时候，我推荐的安装方法会不会也不可以用了，建议要用的时候直接Google，使用国内可以安装的方法

### 设置网络

1. 将网卡混杂模式打开
```shell
sudo ip link set enp3s0 promisc on
```
混杂模式（Promiscuous Mode）是一种网络接口卡 (NIC) 操作模式。在这种模式下，网络接口卡会接收它所在网络上的所有数据包，而不仅仅是发送给它的数据包。

2. 创建 Docker 网卡
```shell
docker network create -d macvlan --subnet=192.168.10.0/24 --gateway=192.168.10.1 -o parent=enp3s0 macnet
```

`macvlan` 是一种网络驱动程序，它允许Docker 容器直接连接到宿主机的物理网络。使用`macvlan` 时，每个容器将获得一个唯一的 MAC 地址， 并直接出现在物理网络上， 就像它是一个独立的设备

### 下载openwrt 镜像并启动容器
[GitHub 仓库](https://github.com/SuLingGG/OpenWrt-Docker) 这里包含了很全的镜像文件包含了 arm，x86，x64的docker镜像，我的机器是NUC，所以我安装的是x64 的 docker 镜像。可以通过`uname -a ` 查看
 ![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/06/f4829c2b1f1eec2cb492ed77ea7253cc.png)

拉取镜像`docker pull sulinggg/openwrt:x86_64`
如果拉取不下来的话，可以使用作者的 阿里云镜像仓库 `registry.cn-shanghai.aliyuncs.com/suling/openwrt:x86_64` 镜像还是一样的

####  启动 OpenWRT 镜像
```shell
docker run --restart always --name openwrt -d --network macnet --privileged sulinggg/openwrt:x86_64 /sbin/init
```

### 设置容器ip
到目前为止， 容器已经启动了，可以通过`docker ps -a`  查看运行的容器，然后查看OpenWRT 的 container ID

我的容器ID是: `e599eb6c7a3a` ，然后我们进入容器内部并设置容器的ip。通过命令 `docker exec -it e599eb6c7a3a` 进入容器内部
```shell
docker exec -it e599eb6c7a3a /bin/bash
```

下面开始设置容器的ip 以便我们能通过ip 地址访问容器，通过命令: `vim /etc/config/network` 来设置ip。
```shell
config interface 'lan'
        option ifname 'eth0'
        option proto 'static'
        option netmask '255.255.255.0'
        option ip6assign '60'
        option ipaddr '192.168.10.111'
        option gateway '192.168.10.1'
        option dns '192.168.10.1'
```

其中的 `option ipaddr` 是你的 `openwrt` 的地址， 注意不要与局域网其他设备冲突。`option gateway` 与 `option dns` 设置你路由器的地址

保存完成后通过命令 `/etc/init.d/network restart ` 重启网络，重启完成后就可以通过IP 访问我们的 OpenWRT 服务了
就可以愉快的使用OpenWRT 了

### 设置OpenWRT
1. 把**桥接网络关闭** 打开网络 -> 接口 -> LAN -> 物理设置 -> 桥接接口 取消勾选

### 更新PassWall 以使用Hysteria 2 和 TUIC

#### 下载最新 Passwall 及 组件包

[openwrt-passwall Github 发布页](https://github.com/xiaorouji/openwrt-passwall/releases)

[依赖包luci-lua-runtime](https://github.com/xiaorouji/openwrt-passwall/issues/2786)
如果出现sing-box 一直安装不上的话，可以将这个依赖包上传并安装好

- 软件包 luci-23.05_luci-app-passwall_4.77-6_all.ipk
- 中文包 luci-23.05_luci-i18n-passwall-zh-cn_git-24.152.54078-47d7784_all.ipk
- 对应CPU架构的组件包 passwall_packages_ipk_x86_64
- [依赖包luci-lua-runtime](https://github.com/xiaorouji/openwrt-passwall/issues/2786) OpenWrt 23.05-rc3 以下版本需要

#### 查询软路由CPU架构
```shell
uname -m
```

#### 安装
```shell
# 进入安装包所在目录  
cd /tmp/upload  
# 查看目录下所有文件  
ls  
# 解压安装包  
unzip luci-lua-runtime_all_fake.zip 
unzip passwall_packages_ipk_x86_64.zip  
# 清屏  
clear  
# 查看目录下所有文件  
ls  
# 安装所有软件包  
opkg install *.ipk --force-reinstall
```

> 参考博客:
> https://augustdoit.men/passwall-hysteria2/
> 