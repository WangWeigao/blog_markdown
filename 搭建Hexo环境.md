---
title: 使用Hexo搭建技术博客
author: w-g-w
date: 2018-12-12 17:08:00
categories: Hexo
tags: [Hexo, Blog]
typora-root-url: ./
---

  

本文最终的实现效果为:

写markdown文件 -> 推送到Github -> 网站自动更新

<!-- more -->

# 使用Hexo搭建技术博客

## 环境

>OS: ubuntu 18.04
>
>Web Server: nginx





## 安装 Node.js

```bash
apt install npm
```



## 安装 Hexo

### 安装Hexo

```bash
npm install hexo-cli -g
hexo init blog_markdown
cd blog_markdown
npm install
hexo server
```

### 给github库建立软链接

```bash
cd /home/ubuntu/blog_markdown/source
mv _posts _posts.bak
ln -s /usr/local/src/blog_markdown .
```



### 实时生成静态文件

```bash
hexo generate -w >/dev/null 2>&1 &
```



## 配置nginx

### 添加配置文件

```nginx
server {
    listen 80;
    server_name blog.7python.com;
    return 301 https://$host$request_uri; 
}

server {
    listen 443 ssl http2;
    server_name blog.7python.com;
    root /home/ubuntu/blog_markdown/public;


    charset utf-8;


    location = /robots.txt  { access_log off; log_not_found off; }

    location ~ /\.(?!well-known).* {
        deny all;
    }

    location = /favicon.ico { access_log off; log_not_found off; }

    # 此处为了与本地图片文件夹对应
    location /images/ {
        root /usr/local/src/blog_markdown/;
    }
}

```

### 配置 https ssl 证书

使用[cerbot-auto](https://certbot.eff.org/)来自动生成证书

官方网站上有详细介绍, 按步骤操作即可

![certbot-auto](/images/certbot-auto.png)

> 上图只截取了部分内容, 具体操作见[官网](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx)

## 建立 GitHub库

### 建库

1. 在github网页界面建立新的repository: **blog_markdown**

### 将代码 Clone 到本地

并且 clone 到本地, 将本地公钥在github上配置好

```bash
cd /usr/local/src
sudo git clone git@github.com:your-github-name/blog_markdown.git
```



>  git使用参见[廖雪峰写的git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

## 安装Hexo主题

- 安装流行的 **next** 主题

  [官网参考](http://theme-next.iissnan.com/getting-started.html)



## 配置webhook

![添加webhook](/images/配置github的webhook.png)

> 上图是**GitHub**中webhook配置的位置

![填写Webhook配置](/images/webhook配置填写.png)

> 上图中的地址要按照自己服务器的实际地址填写



## 安装 webhook 服务器

### 安装 webhook 服务

```bash
apt install webhook
```

### 添加配置文件

> /etc/webhook.conf

```json
[
  {
    "id": "redeploy-webhook",
    "execute-command": "/home/ubuntu/webhook_command_after_push.sh",
    "command-working-directory": "/usr/local/src/blog_markdown/"
  }
]
```

### 添加脚本

> 添加 github 库推送成功后执行的脚本 **webhook_command_after_push.sh**

```bash
#!/bin/bash
git pull
```

上述代码只有一条! 只为拉取最新代码『markdown文件』

### 启动服务

```bash
systemctl start webhook
```

## 测试

### 查看 webhook 推送记录

- 在本地添加新的markdown文件**『如: test.md』**, 并且推送到github
- 查看**webhook**的执行记录

![webhook执行记录](/images/webhook执行记录.png)

> 如果推送记录显示成功, 则说明请求webhook server端成功



### 查看本地是否获取新的markdown文件

![查看本地是否获取最新文件](/images/查看本地是否获取最新文件.png)

### 查看网站是否更新

> 如我的个人博客: https://blog.7python.com