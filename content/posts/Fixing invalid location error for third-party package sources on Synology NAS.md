+++
title = '群晖添加第三方套件源提示无效位置的解决方法'
date = 2023-09-14T21:05:30+08:00
draft = false
categories = ["Tech"]
tags = ["Synology"]
+++

# ***\*群晖添加第三方套件源提示无效位置的解决方法\****

<!--more-->

1、最近由于CA根域名证书过期的原因，造成很多群晖证书失效，最直接的影响就是添加第三方套件源会提示 无效的位置 的

2、因此，我们只需要给群晖更新一下根证书，在电脑的浏览器打开这个

[https://curl.se/ca/cacert.pem](https://curl.se/ca/cacert.pem)

3、系统会自动下载一个证书文件，保存在电脑上面

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272024044.png)

4、找到刚才下载的文件

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272033888.png)

5、打开群晖File Station，在非中文的共享文件夹建立一个子文件夹

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272033766.png)

6、上传文件，用覆盖模式

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272033488.png)

7、把下载到电脑的文件，上传到群晖
![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272034157.png)

8、点中文件，右键属性

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272034270.png)

9、在”位置”后面的内容全部选中—复制—关闭

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272035360.png)
(/volume2 就是储存空间2 )

10、打开群晖控制面板—任务计划—新增—触发的任务—用户定义的脚本

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272037440.png)

11、任务名称随便写，我写的是’cacert’，用户账号选’root’

12、把脚本复制一下(把/volume2/downloads/1/cacert.pem 改成9步骤中复制的)，根据实际路径修改好，粘贴放到下面的位置后，然后点一下确定

```yaml
cp /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt.bak
cp /volume2/downloads/1/cacert.pem /etc/ssl/certs/ca-certificates.crt
```

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272037249.png)

13、在任务列表找到刚刚添加的任务—右键—运行

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272037227.png)

14、点“是”

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272038501.png)

15、再回到群晖的套件中心，在 套件中心—设置—来源—新增，加入第三方套件的地址

https://packages.synocommunity.com/

16、在套件中心—设置—在”允许安装以下发现者发布的套件;“—选择”任何发行者”,![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309272038466.png)
