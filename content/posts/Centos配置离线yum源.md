+++
title = 'Centos配置离线yum源'
date = 2023-09-25T10:48:57+08:00
draft = false
categories = ["Notes"]
tags = ["笔记"]
+++


我们在Llinux系统中安装插件一般都是使用yum来进行安装插件(也可以通过rpm包来安装但是我们现在用的最多的是yum安装)，yum安装可以联网安装和离线安装，离线安装就是不能连通外网的时候来进行安装插件

<!--more-->

**开始:**

1.登录虚拟机

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309281322305.png)



```shell
cd /etc/yum.repos.d
```

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309281322295.png)

可以看到有很多文件我们都不需要这些文件，可以通过sd```mv命令来剪切走

```shell
mv /etc/yum.repos.d/* /opt
```

这个时候进入`yum.repos.d`这个文件夹就会发现东西没有了

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309281324081.png)

这个时候我们输入

```shell
vi local.repo   //这个local这个名称是随便的可以不叫loacl但是后面结尾一定是.repo
输入完这个命令就会到这个界面
```

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309281325091.png)

这个时候按一下i键进入编辑模式输入

```shell
[centos]   //这个地方也是不一定要叫centos也可以叫别的比如[aaa]
name=centos  //这个地方是也name=加你想加的
baseurl=file:///mnt/cd  //这个mnt/cd是你的挂载点0
enabled=1   //这里就按照这个打就好了，意思是开启
gpgcheck=0  //这里也是按照这个打，意思是关闭开机自启动
然后按Esc键输入:wq进行保存并退出
```

然后我们创建我们上面配置文件打的那个/mnt/cd这个目录

```shell
mkdir /mnt/cd
```

然后我们进行挂载

```shell
mount /dev/sr0 /mnt/cd  //这个是将光盘挂载到/mnt/cd这个目录上
```

在我们使用/dev/sr0这个的时候要看看VMware中这个是否连接

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309281325181.png)

挂载成功的命令反馈是

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309281325099.png)

然后我们试着yum安装一个插件

```shell
yum -y install vim
```

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309281325156.png)

出现这个就是成功了