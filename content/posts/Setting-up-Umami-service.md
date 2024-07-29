+++
title = '搭建Umami服务'
date = 2024-07-26T00:10:04+08:00
draft = false
categories = ["Tech"]
tags = ["Tools", "Umami", "Notes"]
+++

### 什么是Umami

Umami 是一种简单、快速、注重隐私的开源分析解决方法。Umami 是 Google Analytics 的最佳替代品， 因为它能让您完全控制数据，而且不会侵犯用户隐私。

### 安装

我使用的安装方法是 **Docker** ， 以下是Umami 的 **docker-compose.yml** ([Dockerfile仓库地址](https://github.com/umami-software/umami/blob/master/docker-compose.yml))
```yaml
---
version: '3'
services:
  umami:
    image: ghcr.io/umami-software/umami:postgresql-latest
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://umami:umami@db:5432/umami
      DATABASE_TYPE: postgresql
      APP_SECRET: replace-me-with-a-random-string
    depends_on:
      db:
        condition: service_healthy
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "curl http://localhost:3000/api/heartbeat"]
      interval: 5s
      timeout: 5s
      retries: 5
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: umami
      POSTGRES_USER: umami
      POSTGRES_PASSWORD: umami
    volumes:
      - umami-db-data:/var/lib/postgresql/data
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
volumes:
  umami-db-data:
```

然后运行
```shell
docker-compose up -d 
```
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/0f7741513ef509edf0636f551872648d.png)

然后访问网页(因为我的服务器3000端口给占用了，所以我改到3333)
默认用户名: `admin` 默认密码: `umami` (登录上去记得修改)
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/07/f387baca6310124e52b63601bc64e593.png)

### 使用
后面的使用步骤和在Umami官网上的使用步骤是一样的，只不过要把`Tracking code`更换一下。