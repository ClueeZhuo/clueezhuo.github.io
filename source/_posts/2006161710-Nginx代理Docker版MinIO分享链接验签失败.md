---
title: 2006161710@Nginx代理Docker版MinIO分享链接验签失败
categories:
  - NGINX
tags:
  - NGINX
  - MinIO
  - DOCKER
date: 2020-06-16 17:09:50
---
# 问题：Nginx代理Docker版MinIO查看分享链接提示延签失败

`nginx minio The request signature we calculated does not match the signature`

# 解决：调整Nginx配置

```ini
upstream minio_svr  {
    server      127.0.0.1:59001;
}

server  {
    listen          80;
    server_name     dbsvr-minio.cluee.tech;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;

        proxy_connect_timeout  300;
        proxy_http_version 1.1;
        proxy_set_header Connection "";

        proxy_pass  http://minio_svr;
    }

    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root        /usr/share/nginx/html;
    }
}
```