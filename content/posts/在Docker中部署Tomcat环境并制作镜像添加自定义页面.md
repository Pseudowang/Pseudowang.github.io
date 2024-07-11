+++
title = '在Docker中部署Tomcat环境并制作镜像添加自定义页面'
date = 2024-05-03T21:39:25+08:00
draft = false
categories = ["Tech"]
tags = ["Notes", "Basic"]
+++

当你开始这个任务的时候，希望你已经基本了解Docker是什么东西，基本的命令

[Docker 快速入门文档(推荐)](https://docker.easydoc.net/doc/81170005/cCewZWoN/lTKfePfP)

[Docker 快速入门文档配套视频](https://www.bilibili.com/video/BV11L411g7U1/)

[Docker官网给Docker的概述(开翻译插件去看)](https://docs.docker.com/get-started/overview/)


### 任务8
> 编写一个Java Web项目，发布为war包，并在Docker容器中运行

我已经写好了一个基本的Java Web 页面，简单的Hello world，并完成了war包的打包
[打包教程CSDN](https://blog.csdn.net/weixin_44741023/article/details/119298059)

#### 安装Docker Engine
我们用的系统是Centos7，所以就可以直接去Docker 官方去找安装教程[Centos Docker安装教程](https://docs.docker.com/engine/install/centos/)，但是我给你的虚拟机已经配置好了Docker环境，所以这一步可以跳过，但是最好可以去看一下了解一下Docker大致的安装步骤，也很简单官方都是直接给一个 安装脚本，复制粘贴跑就完了

#### 编写一个Java Web项目，发布为war包
制作的 web 包已经一起发给你了，名字为`myweb.war`

#### 在Docker 运行一个Tomcat容器
```shell
[root@tomcat ~]# docker run -d -p 8080:8080 tomcat:latest

# -d, --detach  在后台运行container并打印container ID
# -p, --publish 将container端口发布到host 
#tomcat:latest 表示使用tomcat最新的镜像,latest的英文意思就是最新的,最近的
``` 
container 就是容器的意思

![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/cf50489bbbaf281bb970cc6071df1967.png)
你可能觉得上面是报错了，但其实不是的，他们提示的是`Unable to find image 'tomcat:lastest' locally`，说明你本地没有`tomcat`的镜像，他从`Docker registries`中给你拉取就是下载镜像(Docker registries 就像我之前给你解释的应用商城一样，而Docker registries是 Docker 官方的命名)

![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/d5d83b108537df7172c5d2a9884370b3.png)
ChatGPT翻译
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/39daa34325ec3bde89427906cf64a820.png)


如果它报错
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/f9a0b7acba823804d8416965c2ed5ac5.png)
```shell
docker: Error response from daemon: driver failed programming external connectivity on endpoint naughty_torvalds (0edd856e1b25bbf689ba039aac0d2aa33d4eb334fbd50d4915a774912ad27f3e): Error starting userland proxy: listen tcp4 0.0.0.0:8080: bind: address already in use.
```
说明本机的8080 端口给占用了，可以使用`netstat` 来进行查看
```shell
[root@tomcat docker]# netstat -antup | grep 8080                                                                                                  
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      943/nginx: master p
```
看到的是nginx 服务占用了我们的8080端口，然后nginx  服务的pid 为 943，我们使用`kill` 命令来杀掉这个进程
```shell
[root@tomcat docker]# kill -9 943
# -9 为强制删除，强制删掉这个进程
```

以防等会重启的时候又要重新杀掉进程，可以使用`stop`命令和`disable` 来关闭 `nginx`服务，在关闭之前需要确认这个服务是否需要，是否需要一直运行着(这台虚拟机是不需要这个服务的)
```Shell
[root@tomcat Share]# systemctl stop nginx                                
#这会停止nginx服务，但机器重启之后nginx服务还会打开,这个时候就要使用disable命令

[root@tomcat Share]# systemctl disable nginx  #关闭开机自启动                                                    
Removed symlink /etc/systemd/system/multi-user.target.wants/nginx.service.
```

然后我们再重新运行`docker run -d -p 8080:8080 tomcat:latest`

我们使用`docker ps -a ` 查看正在运行的进程，其中的`ps` 代表 process status(进程状态)的缩写

我们使用IP 加端口号8080访问我们的Tomcat网页，查看本机IP可以使用`ip add` 来进行查看
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/f99d33add8f2d0628270705effed0ba7.png)
Tomcat运行成功

#### 在VMware上给Centos建立共享文件夹
1. 打开VMware，打开你正在运行的虚拟机，右键虚拟机
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/9e13971c3d9641849709743bfbc273f7.png)
2. 点击 `设置`
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/73c1e947fce46b5c1211a6b2f65fd4fc.png)
3. 点击`共享文件夹` --> 将`文件夹贡献`选择为`总是启用` --> 点击`添加`，将你要共享的文件夹添加上去(这里要添加上去的文件夹为你存放war包的文件夹) --> 然后就点击 `确认`
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/862c096cb2cdcc087a80933733a65b75.png)
4. 共享文件夹存放的目录是在`/mnt/hgfs` 之中，但是我这边找不到这个文件夹，我在论坛中找到了解决方法[StackExchange](https://unix.stackexchange.com/questions/436723/shared-folder-does-not-appear-in-guest-centos-windows-host-using-vmware)
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/6a1773d7c79649a8bf9ab17612164eed.png)
5. 运行完上面的步骤，就可以在`/mnt/hgfs` 中找到你的共享文件夹了
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/977b9a8e9ecf9f716560d98ae384fc5b.png)
#### 将共享文件夹中的war包发布到Docker中的Tomcat容器中
使用 `docker ps -a` 来查看正在运行的(process status 程序状态) 容器，找到我们刚刚跑起来了Tomcat容器，找到它的`CONTAINER ID`(容器ID)，`-a` 就是 `all` ，就是查看全部容器的状态，这里的容器ID为`279b8b4a2456`
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/e40cfb5a6ab8a77cefd44d14c9212320.png)

进入我们存放 war 包的位置，我这里是`/mnt/hgfs/Share`，因为我没有把war包从共享文件夹中移出来，但是最好是把它从共享文件夹中移出来
```shell
[root@tomcat ]# mkdir /root/packets #创建一个名为packets的文件夹
[root@tomcat ]# cp /mnt/hgfs/Share/myweb.war /root/packets/
#将/mnt/hgfs/Share/文件夹中的myweb.war文件复制到/root/packets 文件夹中

[root@tomcat Share]# cd /root/packets/
#进入/root/packets 这个文件夹
```

使用`docker cp` 命令将war 包复制到容器内
```shell
[root@tomcat packets]# docker cp /root/packets/myweb.war 279b8b4a2456:/usr/local/tomcat/webapps  

# 279b8b4a2456 就是我们Tomcat容器的ID,/usr/local/tomcat/webapps,就是Tomcat容器内存放网页的文件夹

Successfully copied 4.1kB to 279b8b4a2456:/usr/local/tomcat/webapps
```

使用`IP:8080/war包名称/`来访问网站(不知道自己IP地址可以通过ip add 来进行查看)
这台机子的IP 是 192.168.30.40
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/b1903b74f9852d85cdb355fd5d1076cd.png)
成功访问!!

### 任务9
> 用编写Dockerfile方式制作一个 Tomcat 容器，并发布一个war包测试。

#### 制作镜像
```shell
[root@tomcat ~]# mkdir /root/docker  #创建一个文件夹
```

将`apache-tomcat-7.0.96.tar.gz`和`jdk-8u211-linux-x64.tar.gz` 放到你之前创建的共享文件夹中，方便移到虚拟机中
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/2f435ebf4676acef727816f2a182ff77.png)

将`apache-tomcat-7.0.96.tar.gz`和`jdk-8u211-linux-x64.tar.gz`  放到刚刚创建的`docker` 文件夹中
```shell
[root@tomcat Share]# cp /mnt/hgfs/Share/apache-tomcat-7.0.96.tar.gz /root/docker/                                                                                                      
[root@tomcat Share]# cp /mnt/hgfs/Share/jdk-8u211-linux-x64.tar.gz /root/docker/
```

进入到 `docker` 文件夹中
`cd /root/docker`

创建一个 Dockerfile 文件，里面写入
```Dockerfile
# 基本镜像
FROM centos
# 把你上传的 JDK 放到 Docker 容器里面的 root 目录下
ADD jdk-8u211-linux-x64.tar.gz /root
# 把你上传的 Tomcat 放到 Docker 容器里面的 root 目录下
ADD apache-tomcat-7.0.96.tar.gz /root
# 设置环境变量
ENV JAVA_HOME /root/jdk1.8.0_211
# 设置环境变量
ENV CLASSPATH ${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
# 设置环境变量
ENV CATALINA_HOME /root/apache-tomcat-7.0.96
# 设置环境变量
ENV CATALINA_BASE /root/apache-tomcat-7.0.96
# 设置环境变量
ENV PATH ${PATH}:${JAVA_HOME}/bin:${CATALINA_HOME}/bin
# 执行 startup.sh 并打开日志
ENTRYPOINT /root/apache-tomcat-7.0.96/bin/startup.sh && tail -F /root/apache-tomcat-7.0.96/logs/catalina.out

```

使用`docker build -t tomcattest:latest . ` 来构建镜像
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/1625b88414c55b1cf1b2f803015fdfd1.png)

构建完成
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/0c890711c8da5778323bf465bcf59386.png)

使用`docker images` 查看一下刚刚构建好的镜像
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/cf5ecdbf37c3fd2a7701aab4d08d55c2.png)
其中的 `tomcattest` 就是我们刚刚构建好的镜像

#### 使用刚刚构建好的镜像运行容器
```shell
[root@tomcat docker]# docker run -d -p 8090:8080 tomcattest
#因为之前的已经使用了8080端口，这次我们使用8090端口

f69050d2fdb525ac864bef391db51283a14ad7afbcc78ec719bb254e7f5df9ec
```

使用`IP:8090` 来访问一下Tomcat
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/e9ff8322cad2c432d3bdcc79d3dbbffd.png)
访问成功！

#### 将war包上传到tomcat容器之中

参考任务8的上传war包到容器之中

```shell
[root@tomcat packets]# docker cp /root/packets/myweb.war f69050d2fdb5:/root/apache-tomcat-7.0.96/webapps/   
# f69050d2fdb5 就是刚刚构建的镜像运行的tomcat的容器id，可以通过docker ps -a 查看

Successfully copied 4.1kB to f69050d2fdb5:/root/apache-tomcat-7.0.96/webapps/
```

使用 `IP:8090/war包名/`来进行访问
我们的war包名字为 `myweb` 

![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/4b3ef885825135e367fdc5d4fdfb92fa.png)
访问成功~！！