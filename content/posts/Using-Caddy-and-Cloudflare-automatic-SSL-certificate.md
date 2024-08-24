+++
title = '使用Caddy和Cloudflare自动获取SSL证书'
date = 2024-07-29T00:07:48+08:00
draft = false
categories = ["Tech"]
tags = ["Caddy", "Basic", "Cloudflare"]
+++

> 由于之前为博客网站搭建了 Umami 统计工具，并计划将 Vaultwarden 服务从 Homelab 迁移到服务器上运行，以便在外网访问自己的 Vaultwarden 服务，我开始学习使用 Caddy 来获取 SSL 证书。通过将网站绑定到我在 Cloudflare 上的域名中，我可以更方便地管理和访问这些服务。

我的服务器系统为：`Debian 12`

## Caddy
这是 [Caddy](https://caddyserver.com/)的官网
### 安装Caddy
安装该软件包后， Caddy 会作为名为 `caddy` 的 systemd 服务自动启动和运行
稳定版本: 
```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```
如果执行命令有报错的话，不妨先执行一下`apt-get update`先

然后我们运行
```bash
caddy
```
没有报错的话就是安装成功了

### 运行Caddy
`caddy run` 和 `systemctl start caddy` 是启动 Caddy 的两种不同方法

1. `caddy run`： 通常用于开发、调试或测试环境中，因为她会直接将输出打印到终端， 便于查看日志和调试信息。也不会将 Caddy 当作系统服务来管理， 它只是在当前会话中启动 Caddy 可以通过`Ctrl + C` 停止

2. `systemctl start caddy`： 会将 Caddy 作为一个系统服务来启动。通过 systemctl 启动的服务可以由系统自动管理，包括在系统启动时自动启动、记录日志、监控状态等

我这里用的是`caddy run` 以及 `caddy start` 等。Caddy 也有 [Command Line](https://caddyserver.com/docs/command-line)页面来解释每个命令的作用和意思

如果你打算用 Caddy 自带的服务管理框架的时候，你运行`caddy start`或者`caddy run` 的时候可能会碰到一下报错
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/841744f791e0fc832deb0bebec4127fe.png)
我猜测是 `systemctl` 已经启动了 caddy 服务，使用 Caddy 自带的服务管理框架启动的时候，因为`systemctl` 已经启动了caddy服务， 所以将caddy的 API 端口给占用了，所以我们只需要`netstat -tulp | grep 2019`，找到占用的进程号
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/e82fcae684e789a83327616a41dca7ba.png)
然后使用`kill -9 2804` 将它 kill 掉(2804是我的进程号，这里要改成自己的)

这个时候我们再使用`caddy start` 就不会报错咯
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/bf7fdc88b904f712f38ee8bbc3c09d53.png)

然后可以直接通过 IP 在浏览器中访问的到网页
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/33971d68694a709c4e06f8004a6cfd6d.png)

### Caddy的基础配置
进入 caddy 配置文件的目录
```bash
cd /etc/caddy
```

#### 更改caddy端口

如果你服务器的80端口和443端口给占用了，可以将caddy的`https`和`http`端口给更改，只需要在`Caddyfile`中添加。[Caddy社区回答](https://caddy.community/t/cannot-change-default-http-and-https-ports/14771/2)
```bash
{                                             
        https_port 8443                                              
        http_port 8080
        local_certs
}
```

### 添加 Module dns.providers.cloudflare
> [NON-STANDARD]此模块不随 Caddy 一同提供。您可以通过使用 xcaddy 或我们的下载页面添加它。非标准模块可能由社区开发，并不受 Caddy 项目的官方支持或维护。

**Custom builds:**[https://github.com/caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare)

**Custom builds：** `xcaddy build --with github.com/caddy-dns/cloudflare`

#### Go的安装
选择好自己的系统，右键复制地址
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/4bee027316a1bca1a2750cbe5d1f9ddf.png)

回到终端中执行
```bash
wget https://go.dev/dl/go1.22.5.linux-amd64.tar.gz #后面为你刚刚在浏览器中复制的地址
```

1. 删除之前的Go 安装文件已经将刚刚下载好的包解压到 /usr/local  
```bash
 rm -rf /usr/local/go && tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
```

2. 将 /usr/local/go/bin 添加到PATH环境变量之中
```bash
export PATH=$PATH:/usr/local/go/bin
```

3. 输入以下命令来验证是否成功安装Go
```bash
 go version
```
#### 安装xcaddy
前提条件: 已安装[Go](https://go.dev/doc/install)

你可以从源代码构建`xcaddy`
```bash
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest
```

对于 Debian， Ubuntu和 Raspbian，可以直接从官方的 [Cloudsmith](https://cloudsmith.io/~caddy/repos/xcaddy/packages/) 仓库获取 `xcaddy` 包 (我选的是这个方法)
```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/xcaddy/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-xcaddy-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/xcaddy/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-xcaddy.list
sudo apt update
sudo apt install xcaddy
```

####  构建 dns.providers.cloudflare 模块
我选择回到`/etc/caddy/` 目录下运行， 因为会生成一个执行文件
```bash
xcaddy build --with github.com/caddy-dns/cloudflare
```
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/dba55acbd1d646241f27adc4a27c6129.png)

### 创建 Cloudflare 的  tokens
1. 登录 Cloudflare 并转到要使用Caddy的域名
2. 向下滑之后右边你会看到标题为 "API" 的部分， 点击  "Get your API token"
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/e82790ea24fe8eeb8c93962fc1662e19.png)
3. 然后点击 “Create Token”
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/654a3048d1f018fe7e65e6bba183fd60.png)
4. 在 “User API Tokens” 页面，滑动到底部， 点击 "Create Custom Token"
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/09b9564762e32ab33f5d60f2343d964a.png)
5. 给你的Token 命名， 然后添加两个 "Permissions"
	1. Zone - Zone - Read
	2. Zone - DNS -Edit
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/d98e18d92240b09caab26b1a04bb92bd.png)
6. 然后点击 “Continue to summary”， 现在就能看到您的API token了，然后将它复制下来
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/a1f80461da93f1d3d0eb13de28a62b88.png)
### Caddyfile编写
可以将`Caddyfile` 文件里面的内容全部注释掉或者删掉，因为我的`Umami`服务是在`3000`端口运行的，所以要`reverse_proxy 127.0.0.1:3000`。

以下是我的Caddyfile(example.com要换成你的域名)

```yaml
example.com {
        reverse_proxy 127.0.0.1:3000
        tls {
                dns cloudflare {env.CLOUDFLARE_API_TOKEN}
        }
}
```

然后在终端中输入之前拿到的Cloudflare API token
```bash
export CLOUDFLARE_API_TOKEN="your-api-token"
```
这个环境变量只在当前的终端会话中有效，一旦你关闭了终端或者退出了登录，这个环境变量就会失效。

如果你希望在每次登录时自动设置这个环境变量，可以将它添加到你的 shell 配置文件中，比如 `.bashrc` 或 `.bash_profile`

### 运行

在`/etc/caddy/`文件夹下执行`./caddy run` 启动(如果有报错大概率是端口冲突，前面有解决方法)。


在 Cloudflare 中添加好记录，就可以尝试访问了
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/e126e7f44f905598709b689be833cb88.png)
访问成功！！


**参考文档**
- [Caddy官方文档](https://caddyserver.com/docs/)
- https://samedwardes.com/blog/2023-11-19-homelab-tls-with-caddy-and-cloudflare/
- https://roelofjanelsinga.com/articles/using-caddy-ssl-with-cloudflare/
- https://developers.cloudflare.com/fundamentals/api/how-to/create-via-api/
- https://caddyserver.com/docs/modules/dns.providers.cloudflare
- https://www.youtube.com/watch?v=WgUV_BlHvj0
- https://github.com/caddy-dns/cloudflare
