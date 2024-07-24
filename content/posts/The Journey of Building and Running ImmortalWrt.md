+++
title = '从官方固件到Docker镜像：ImmortalWrt构建与运行之旅'
date = 2024-07-19T18:28:24+08:00
draft = false
categories = ["Tech"]
tags = ["Docker", "OpenWrt", "Basic"]
+++

> 自从上次因为不小心(手贱)将自己小服务器分区给整坏了，导致开不了机，重新搭建了OpenWrt，不到1个月的时间，这次觉得上次使用的镜像太久没有更新了，然后加上是别人编译的镜像，这次想着自己来做一次镜像，并且学习一下，也碰到了很多自己没有遇到过的报错，也在一次又一次的报错中学习到了很多东西，如果有写错的地方，欢迎评论提出或者邮件，谢谢！

### 使用官方固件
这里以`x86-64`平台为例

首先获取固件的下载地址
1. 打开[ImmortalWrt的官网](https://downloads.immortalwrt.org/)， 选择当前最新的稳定版本`23.05.3`(笔记后面用的是23.05.2，因为写笔记的时候 23.05.3 刚刚更新，有些导航页还没完成)
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/72122da509ff4e509bd6ee126a593cfa.png)

2. 选择`x86`平台

![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/3b715cc258e5faa3b55806ce62f893aa.png)

3. 选择`64`位
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/5fbca3c211d9dd9c944acd9efdb11d11.png)

4. 选择固件`rootfs.tar.gz`
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/d654854992327e2382b6a36c56074b0c.png)

5. 鼠标右键点击"复制链接地址"获取到固件的下载地址，后面第6步需要用到
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/914a169410dc5dc6e62e981d3bebda1b.png)

6. 开始构建镜像
```shell
git clone https://github.com/crazygit/openwrt-x86-64.git openwrt-x86-64
cd openwrt-x86-64
# 参数1: 第5步中获取的固件下载地址
# 参数2: docker镜像的名字，可以随便指定: 如pseudowang/openwrt-x86-64
./build.sh "https://downloads.immortalwrt.org/releases/23.05.2/targets/x86/64/immortalwrt-23.05.2-x86-64-rootfs.tar.gz" pseudowang/openwrt-x86-64
```
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/de260c14630270aa732ee047115e2a51.png)

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

### 使用刚刚创建的镜像并启动容器

```shell
docker run --restart always --name openwrt -d --network macnet --privileged pseudowang/openwrt-x86-64
```

#### 设置容器ip
到目前为止， 容器已经启动了，可以通过`docker ps -a`  查看运行的容器，然后查看OpenWRT 的 container ID

我的容器ID是: `48ec9c884f45`， 然后我们进入到容器内部并设置容器的ip。通过命令`docker exec -it 48ec9c884f45 /bin/sh` 进入容器内部
```shell
docker exec -it 48ec9c884f45 /bin/sh
```

下面开始设置容器的ip 以便我们能通过ip 地址访问容器，通过命令: `vi /etc/config/network` 来设置ip
```shell
config interface 'lan'
        option ifname 'eth0'
        option proto 'static'
        option netmask '255.255.255.0'
        option ip6assign '60'
        option ipaddr '192.168.10.110'
        option gateway '192.168.10.1'
        option dns '192.168.10.1'
```

其中的 `option ipaddr` 是你的 `openwrt` 的地址， 注意不要与局域网其他设备冲突。`option gateway` 与 `option dns` 设置你路由器的地址

保存完成后通过命令 `/etc/init.d/network restart ` 重启网络，重启完成后就可以通过IP 访问我们的 OpenWRT 服务了


### 登录网页并下载软件包
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/4fd0a0be3df58c625594823706faa404.png)
一开始是不需要密码登录，直接回车登录就行，登录上去记得要更改密码

#### 下载软件包
系统 --> 软件包， 然后点击 "更新列表"
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/100b117b1aa3b03738d96e6b0e0043a6.png)
但是 这里会有一个报错，因为我们没有安装`wget-ssl` 这个包，所以会导致更新不了，再加上默认的地址似乎不能够使用，所以我换了一个新的地址，这个地址是在官方仓库有人提的[Issues](https://github.com/immortalwrt/immortalwrt/issues/1124)中拿到的。(`wget returened 1 `)
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/bbef87a17d7f992c7c147b9f2942f9e8.png)

因为没有安装`wget-ssl` 这个包，update 不了，所以暂时把`https` 更换为 `http`， 安装源的文件路径为(`/etc/opkg/distfeeds.conf`)
```shell
src/gz immortalwrt_core http://mirrors.cernet.edu.cn/immortalwrt/releases/23.05.1/targets/x86/64/packages
src/gz immortalwrt_base http://mirrors.cernet.edu.cn/immortalwrt/releases/23.05.1/packages/x86_64/base
src/gz immortalwrt_luci http://mirrors.cernet.edu.cn/immortalwrt/releases/23.05.1/packages/x86_64/luci
src/gz immortalwrt_packages http://mirrors.cernet.edu.cn/immortalwrt/releases/23.05.1/packages/x86_64/packages
src/gz immortalwrt_routing http://mirrors.cernet.edu.cn/immortalwrt/releases/23.05.1/packages/x86_64/routing
src/gz immortalwrt_telephony http://mirrors.cernet.edu.cn/immortalwrt/releases/23.05.1/packages/x86_64/telephony
```

1. 更换安装源，并暂时把`https` --> `http` 之后， update 成功
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/f77445328429a633ed2f7bbd82475dd2.png)

![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/0246f6fc402232e181e6ed2bc5579ef5.png)

2. 安装`wget-ssl` 软件包
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/bcbeef01e7f42666d8abac4419db8c48.png)

安装成功后，重新将`http` 恢复为 `https`
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/fde1f8575033b35134b714b132f2b84d.png)

`update` 成功!!!
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/a13897efb800c7cb438b680f057f8128.png)

![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/5ee0d1dc47083e8e564565b8411e5e75.png)

3. 安装`theme-argon` 软件包
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/2a6b98d1162f792c05c924b9ac4393f4.png)

就可以愉快的使用OpenWRT 了。我通过这些步骤完成了ImmortalWrt的搭建，如果你在搭建过程中有错误或者文章哪里写错了，可以在评论中指出，也可以给我发邮件

> 参考网站: https://github.com/crazygit/openwrt-x86-64
