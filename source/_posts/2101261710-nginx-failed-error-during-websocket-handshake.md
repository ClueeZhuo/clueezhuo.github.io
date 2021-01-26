---
title: '2101261710@nginx_failed_error:during_websocket_handshake'
categories:
  - NGINX
tags:
  - WEBSOCKET
date: 2021-01-26 17:14:31
---

### 报错nginx failed error: during websocket handshake

```shell
location / {
proxy_pass http://xxx_svr;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_set_header Host $host;
}
```