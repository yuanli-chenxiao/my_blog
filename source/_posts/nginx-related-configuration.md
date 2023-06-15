---
title: Nginx配置
date: 2023-06-15 19:38:35
tags:
    - nginx
categories: 
    - nginx
description: nginx配置接口、静态文件
---

## nginx配置api、static

```ini
location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ =404;
}

location ^~ /api {
    proxy_pass http://127.0.0.1:8000/api;
    proxy_redirect off;

    proxy_read_timeout 350s;
    proxy_set_header X-Forwarded-Proto      $scheme;
    proxy_set_header Host                   $http_host;
    proxy_set_header X-Forwarded-For        $proxy_add_x_forwarded_for;
}

location ^~ /static/upload_file {
    proxy_pass http://127.0.0.1:8000/static/upload_file/;
    proxy_redirect off;

    proxy_read_timeout 350s;
    proxy_set_header X-Forwarded-Proto      $scheme;
    proxy_set_header Host                   $http_host;
    proxy_set_header X-Forwarded-For        $proxy_add_x_forwarded_for;
}

location ^~ /static {
    proxy_pass http://127.0.0.1:8000/static;
    proxy_redirect off;

    proxy_read_timeout 350s;
    proxy_set_header X-Forwarded-Proto      $scheme;
    proxy_set_header Host                   $http_host;
    proxy_set_header X-Forwarded-For        $proxy_add_x_forwarded_for;
}
```
