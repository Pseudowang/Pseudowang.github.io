+++
title = '在服务器部署RSS服务'
date = 2023-09-14T21:05:37+08:00
draft = false
categories = ["Notes"]
tags = ["工具"]
+++


### 步骤:

系统为：Ubuntu 20.04

1.  安装Docker Engine
2.  部署RSS服务

<!--more-->

### 1. 安装Docker Engine

使用apt存储库进行安装

#### 设置存储库

1.  更新软件包索引并安装`apt`软件包以运行`apt`通过HTTPS使用存储库:

    ```shell
    $ sudo apt-get update
    $ sudo apt-get install ca-certificates curl gnupg
    ```

2.  添加Docker 的官方 GPG 密钥

    ```shell
    $ sudo install -m 0755 -d /etc/apt/keyrings
    $curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    $sudo chmod a+r /etc/apt/keyrings/docker.gpg
    ```

3.  使用一下命令设置存储库

    ```shell
    $ echo \
      "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```



#### 安装Docker Engine

1.  更新`apt`包索引:

    ```shell
    $ sudo apt-get update
    ```

2.  安装Docker Engine、containerd 和Docker Compose

    要安装最新版本，运行

    ```shell
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

3.  通过运行`hello-world`镜像验证 Docker Engine 安装是否成功

    ```shell
    $ sudo docker run hello-world
    ```

    此命令下载测试镜像并在 container 中运行它。当 container 运行时，它会打印一条确认消息并退出



### 2. 部署RSS服务

#### Docker Compose 部署

1.  下载 [docker-compose.yml](https://github.com/DIYgod/RSSHub/blob/master/docker-compose.yml)

    ```shell
    $ sudo wget https://raw.githubusercontent.com/DIYgod/RSSHub/master/docker-compose.yml
    ```

2.  检查有无需要修改的配置(映射端口啥的)
    ```shell
    $ sudo vi docker-compose.yml
    ```

3.  创建 volume 持久化 Redis 缓存

    ```shell
    $ sudo docker volume create redis-data
    ```

4.  启动

    ```shell
    $ sudo docker-compose up -d 
    ```



#### Docker 部署

1.  运行下面的命令下载 RSSHub 镜像

    ```shell
    $ sudo docker pull diygod/rsshub
    ```

2.  然后运行 RSSHub 即可

    ```shell
    $ sudo docker run -d --name rsshub -p 1200:1200 diygod/rsshub
    ```

    在浏览器中打开 http://IP:1200/ 

    可以使用下面的命令来关闭 RSSHub

    ```shell
    $ sudo docker stop rsshub
    ```



#### 更新 RSSHub

1.  删除旧容器

    ```shell
    $ docker-compose down
    ```

2.  如果之前下载/使用过镜像，下方命令可以帮助你获取最新版本：这可能可以解决一些问题

    ```shell
    $ docker pull diygod/rsshub
    ```

    然后重复安装步骤

3.  删除旧容器

    ```shell
    $ docker stop rsshub
    $ docker rm rsshub
    ```

    然后重复安装步骤



现在RSSHub服务就部署完成

<br>



>  参考文档
>
>  [Install Docker Engine](https://docs.docker.com/engine/install/ubuntu/)
>
>  [RSSHub Docs](https://docs.rsshub.app/)