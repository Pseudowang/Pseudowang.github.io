+++
title = '在服务器上部署简单文件提交系统'
date = 2024-09-16T00:00:31+08:00
draft = false 
categories = ["Tech"]
tags = ["Notes", "Basic", "Python"]
+++

> 为了帮助老师收集所有同学的作业并统一打包发送，我之前一直通过微信收发文件，但这种方式容易导致文件混乱。因此，我决定开发一个网站，让同学们可以直接在网页上上传作业。这个想法源于我之前学习的 CS50 Python 网课，发现使用 Python 开发后端相对简单。随后，我了解了 Flask 框架，并开始根据其官网文档学习开发。这个项目不仅实现了文件上传功能，也帮助我更深入地学习和掌握 Flask 框架。

我的服务器系统是 `Debian 11`

### 搭建 Caddy 服务 并且使用 Cloudflare 自动获取SSL 证书

可以看我之前写的这一篇文章: [使用Caddy和Cloudflare自动获取SSL证书](https://wangzhr.top/posts/using-caddy-and-cloudflare-automatic-ssl-certificate/)

### 搭建文件上传环境

[Homework-Upload-System](https://github.com/Pseudowang/Homework-Upload-System.git) 这是我使用 `Python` + `html` 写的简单网页，可以进行拉起更改

只需要给`.env` 文件添加上 缤纷云桶的 `access key` 和 `secret key`

#### 搭建Python 环境

1. 安装Python环境

```bash
sudo apt update
sudo apt install python3 python3-pip
sudo apt install python3-venv
```
因为要使用虚拟化环境。所以需要安装`python3-venv` 包

然后进入你的刚刚从 GitHub 仓库拉起的代码 的文件目录，我的目录是`/uploadsys`，以下是我的目录结构
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/09/baa8de215faf78e93ab9330282a7fc99.png)

2. 然后`/uploadsys`目录下创建虚拟环境并激活它
```bash
cd /uploadsys/
python3 -m venv myvenv
source myvenv/bin/activate
```

3. 安装`requirements.txt` 中的依赖包
```bash
pip install --upgrade  pip
```

如果您的系统是 `Debian 11` 然后是  **Python 3.9**  的话可能会出现一个错误，关于`urllib3` 的
```
INFO: pip is looking at multiple versions of botocore to determine which version is compatible with other requirements. This could take a while. ERROR: Cannot install -r requirement.txt (line 3) and urllib3==2.2.2 because these package versions have conflicting dependencies. The conflict is caused by: The user requested urllib3==2.2.2 botocore 1.35.13 depends on urllib3<1.27 and >=1.25.4; python_version < "3.10" To fix this you could try to: 1. loosen the range of package versions you've specified 2. remove package versions to allow pip to attempt to solve the dependency conflict ERROR: ResolutionImpossible: for help visit https://pip.pypa.io/en/latest/topics/dependency-resolution/#dealing-with-dependency-conflicts
```
我之前在`Debian 12` 的系统环境安装的是 **Python 3.11** 因为在  **Python 3.10**  及更高的版本中，`botocore` 和 `boto3` 不再限制 `urlib3` 的版本，允许使用较新的 `urllib3`版本。而 **Python3.9** 及以下版本中，`botocore 1.35.13` 限制了 `urllib3` 的版本为 `<1.27`。这就与我的 `requirements.txt` 文件中的 `urllib3==2.2.2` 发生了冲突，我这里直接限制了 `urllib3` 的版本，限制为 `urllib3>=1.25.4,<1.27`，只需要更改 `requirements.txt` 文件中的 `urllib3` 版本就ok了


4. 安装 Gunicorn 发布应用
```bash
pip install gunicorn
```

使用 Gunicorn 运行你的 Flask 应用。我这里的 Flask 应用文件名为 `app.py`，然后发布到指定的 9876 端口
```bash
gunicorn app:app --bind 0.0.0.0:9876
```
然后就可以根据之前写的 [使用Caddy和Cloudflare自动获取SSL证书](https://wangzhr.top/posts/using-caddy-and-cloudflare-automatic-ssl-certificate/) 中的方法直接解析到我的域名之上了
