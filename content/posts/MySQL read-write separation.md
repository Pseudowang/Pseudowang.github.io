+++
title = 'MySQL读写分离'
date = 2023-09-14T21:02:55+08:00
draft = false
categories = ["Tech"]
tags = ["MySQL"]
+++




### MySQL基本教程

#### 增加用户`phpadmin`并完成授权

1.  创建数据库用户`phpadmin`可以从网站服务器(192.168.30.21)登录数据库服务器

    ```mysql
    create user 'phpadmin'@'192.168.30.21' identified by '000000';
    ```

    

2.  为数据库用户`phpadmin`授权：可以读写store库

    ```mysql
    grant all privileges on store.* to 'phpadmin'@'192.168.30.21'; 
    ```

    

    如果想给store库下的goods表权限的话只需要把*改成goods就可以了，这里的`*`代表这store库下的全部表

    ```mysql
    grant all privileges on store.goods to 'phpadmin'@'192.168.30.21';
    ```

#### 创建Table

在给表中插入数据的前提是创建好表才可以往表中插入数据

1.  创建table(给store 库下创建名为goods的表)

    ```mysql
    create table store.goods(gid char(32),gname char(32),gprice decimal(9,2));
    ```

    

2.  给goods表中插入数据

    ```mariadb
    insert into store.goods values('001','apple','11');
    ```
    
    

### MySQL读写分离

#### 架构图

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309281327149.png)

#### MySQL读写分离概念

MYSQL读写分离的原理其实就是让Master数据库处理事务性增、删除、修改、更新操作（CREATE、INSERT、UPDATE、DELETE），而让Slave数据库处理SELECT操作，MYSQL读写分离前提是基于MYSQL主从复制，这样可以保证在Master上修改数据，Slave同步之后，WEB应用可以读取到Slave端的数据。

MySQL-proxy是官方提供的MySQL中间件产品可以实现负载平衡，读写分离，failover等。

SQL语句并不直接进入到master数据库或者slave数据库，而是进入到 proxy，然后proxy判断这条语句是有关写的语句（包括insert、update、delete）还 是读语句（select），当是写语句的时候，那么proxy将向master所在的服务器发出请 求，同理，如果是读语句的时候，proxy将向slave所在的服务器发出请求。



##### 介绍

1.  处于client段和MySQL server 端之间的应用
2.  可以监测、分析或改变它们的通信
3.  使用灵活，没有限制，常用的用途包括：负载平衡，故障、查询分析，查询过滤和修改等等



#### 环境搭建

需要3台服务器，Proxy可以选择和MySQL部署在同一台服务器，也可以选择单独部署在另一台独立服务器。

并且Master和Slave服务器配置好了数据库的主从同步

![](https://wangzhrbuckets.s3.bitiful.net/picture/2023/09/202309281327455.png)

| hostname | IP            |
| -------- | ------------- |
| Master   | 192.168.30.91 |
| Slave    | 192.168.30.92 |
| Proxy    | 192.168.30.93 |

下载`mysql-proxy`使用[MySQL-Proxy](https://downloads.mysql.com/archives/proxy/?spm=a2c6h.12873639.article-detail.6.3255c98a05cY9H#downloads)

注意：要在Master和Slave数据库里面给phpadmin用户权限

例:`grant all privileges on store.goods to 'phpadmin'@'192.168.30.21';  `  

其中的登录IP为你的MySQL-proxy服务器的IP



### 开始配置MySQL-Proxy

-  使用的服务器系统为RedHat 9



#### 1. MySQL主从同步

老师文档里面有教过，先暂时不写教程



#### 2. MySQL-Proxy配置

1.  安装MySQL-Proxy

    -  解压到指定目录

    ```
    tar -zxvf mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz
    mv mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit /usr/local/mysql-proxy
    ```

2.  配置mysql-proxy

    -  执行命令

    ```shell
    cd /usr/local/mysql-proxy
    mkdir lua #创建脚本存放目录
    mkdir logs #创建日志目录
    cp share/doc/mysql-proxy/rw-splitting.lua ./lua #复制读写分离配置文件
    cp share/doc/mysql-proxy/admin-sql.lua ./lua #复制管理脚本
    ```

    -  配置文件(创建/etc/mysql-proxy.cnf配置文件)

    ```
    [mysql-proxy]
    user=root  #运行mysql-proxy的用户
    admin-username=root  #主从mysql共有的用户
    admin-password=000000 #用户的密码
    proxy-address=192.168.30.93  #mysql-proxy运行ip和端口,不加端口默认4040
    proxy-backend-addresses=192.168.30.91  #指定后端主master写入数据
    proxy-read-only-backend-addresses=192.168.30.92 #指定后端从slave读取数据
    proxy-lua-script=/usr/local/mysql-proxy/lua/rw-splitting.lua #指定lua脚本
    admin-lua-script=/usr/local/mysql-proxy/lua/admin-sql.lua
    log-file=/usr/local/mysql-proxy/logs/mysql-proxy.log
    daemon=true #以守护进程方式运行
    log-level=info  #定义log日志级别，由高到低分别有(error|warning|info|message|debug)
    keepalive=true  #mysql-proxy崩溃时，尝试重启
    ```

    复制一定要删除全部注释，要不然会有报错，最好把注释前的空格也给删了

    然后设置`mysql-proxy.cnf`权限:`chmod 660 /etc/mysql-proxy.cnf`(所属用户和所属组有读写权限)

3.  配置读写分离脚本

    `vi /usr/local/mysql-proxy/lua/rw-splitting.lua`

    ```shell
    if not proxy.global.config.rwsplit then
     proxy.global.config.rwsplit = {
      min_idle_connections = 1, #默认超过4个连接数时，才开始读写分离，改为1
      max_idle_connections = 1, #默认8，改为1
      is_debug = false
     }
    end
    ```

4.  启动MySQL-proxy

```she
/usr/local/mysql-proxy/bin/mysql-proxy --defaults-file=/etc/mysql-proxy.cnf
```

5.  验证是否启动成功

```
netstat -tupln | grep 4000
```

​	关闭命令

```
killall -9 mysql-proxy  #-9 为强制杀掉进程
```



#### 3.  测试

1. 因为Master数据库负责读写，而Slave数据库只可以读

​	在`fm1.php`和`fm2.php`文件中负责连接数据库读取`store`库下的`goods`中的表数据。但我的`Slave`数据库没有`store`库也没有`goods`表，只有我的`Master`数据库才有`goods`表。所以当我通过MySQL-Proxy服务器进行操作的时候，是查不到表中的数据的，因为我们读取数据库是走`Slave`数据库的，但我们`Slave`数据库是没有这个文件的。这就可以证明读取数据走的是`Slave`服务器而不是`Master`服务器。当我们在`Slave`数据库创建出`goods`表，并添加一些数据时候，当我们运行`fm1.php`的时候就可以查询到数据了



2.  暂停读写分离测试
    1.  先暂时停掉主从同步
    2.  在master节点插入数据，然后通过MySQL-proxy查询
    3.  在slave节点插入数据，然后通过MySQL-proxy查询

