---
type: "posts"
author: "jager"
title: "nginx反向代理"
date: "2021-08-03"
tags: [
"nginx",
"反向代理",
]
---

> 利用Nginx的反向代理可以实现多个域名指向同个服务器的不同网站，本文将介绍如何配置

<!--more-->

## nginx反向代理配置


第一个站点
```
server {
    listen 443;
    server_name           www.jayg.xyz;
    ssl                   on;
    ssl_certificate       /home/www/ssl/1_www.jayg.xyz_bundle.crt;
    ssl_certificate_key   /home/www/ssl/2_www.jayg.xyz.key;
    ssl_session_timeout   5m;
    ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers           HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }   
    error_page 500 502 503 504 404 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }   
}
```
第二个站点
```
server {
    listen 443 ssl;
    server_name           www.hawtech.cn;
    ssl_certificate       /home/www/ssl/1_www.hawtech.cn_bundle.crt;
    ssl_certificate_key   /home/www/ssl/2_www.hawtech.cn.key;
    ssl_session_timeout   5m;
    ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers           HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://127.0.0.1:8081;
    }   
    error_page 500 502 503 504 404 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }   
}
```
