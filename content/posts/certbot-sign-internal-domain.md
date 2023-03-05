---
title: "使用 Certbot 对内网域名签名"
date: 2023-03-05T22:48:16+11:00
draft: false
tags:
 - Certbot
 - Domain
categories:
---

# 前情提要

有一个 All-In-~~boom~~One 的服务器，然后为了方便管理，不用记 IP 地址。搭建了一台内网 DNS 服务，并使用自己的某三级域名 internal.example.org 作为内网域名使用。

虽然说内网服务，上不上 HTTPS 没差，但在 iOS Safari 上会有不安全的提醒，以及没有那把小锁🔒，看着不舒服。

Certbot 在签名的时候，会查 DNS 记录，但内网域名只在内网使用，怎么可能会有公网的 DNS 记录呢。

以及，我的 nginx 是放在 Docker 里跑的，相对于直接在宿主机上还是有点区别的

# 解决方案

在某 medium 文章中[^1]，使用了 Certbot 的 Docker 镜像，并且将该容器的一些目录和 nginx 容器目录做了共享。

```Dockerfile
services:
  web:
    container_name: nginx
    image: docker.io/nginx
    volumes:
      - ${PWD}/conf.d:/etc/nginx/conf.d
      - ${PWD}/certbot/conf:/etc/letsencrypt
      - ${PWD}/certbot/www:/var/www/certbot
  certbot:
    image: docker.io/certbot/certbot
    volumes:
      - ${PWD}/certbot/conf:/etc/letsencrypt
      - ${PWD}/certbot/www:/var/www/certbot
```

## 配置 Nginx

然后在 nginx 的 `*.conf` 文件中增加

```
server {
    listen 80;

    location / {
        return 301 https://$host$request_uri;
    }

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
}

server {
    listen 443;

    ssl_certificate /etc/letsencrypt/live/<YOUR-DOMAIN>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<YOUR-DOMAIN>/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    ...
}

```

## 下载配置文件

然后下载下 TLS 的一些配置

```
curl -s https://raw.githubusercontent.com/certbot/certbot/master/certbot-nginx/certbot_nginx/_internal/tls_configs/options-ssl-nginx.conf > "${PWD}/certbot/conf/options-ssl-nginx.conf"
curl -s https://raw.githubusercontent.com/certbot/certbot/master/certbot/certbot/ssl-dhparams.pem > "${PWD}/certbot/conf/ssl-dhparams.pem"
```

## 手动签名[^2]

首先你肯定有二级域名（或者叫一级域名，如果不考虑 `.com` 这一层）

```sh
podman-compose run --rm --entrypoint "\
certbot certonly --manual --preferred-challenges dns \
-d <your-internal-domain>" certbot
```

这里会提示，让你在域名里加一个 TXT 记录，Key 是 `_acme-challenge.<prefix>`，等 TXT 记录生效后，回车继续，签名信息就被写入到了 `${PWD}/certbot/conf/live/<domain>`

这时候再重启 nginx 服务，就解决所有问题了。

## 后续

这么做的一个坏处就是，没法自动更新证书，需要手动跑命令更新。

考虑到证书有效期 3 个月， 到还能接受。

[^1]: [Nginx and Let’s Encrypt with Docker in Less Than 5 Minutes](https://pentacent.medium.com/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71)
[^2]: [「Certbot」- 在内网中申请证书的方法 @20210312](https://www.cnblogs.com/k4nz/p/14523264.html)