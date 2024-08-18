---
title: 2408190105@NGINX+EMQX+WSS
categories:
  - EMQX
tags:
  - SHELL
date: 2024-08-19 01:05:00
---
### EMQX-WSS 18084:8084
``` bash
HOST 18084端口代理容器8084端口，HOST 8084全部让给NGINX
```

### NGINX-conf.d/emqx-wss.cluee.tech.conf
``` bash
server {
    listen 18084 ssl;
    server_name cluee.tech;

    ssl_certificate D:/App/SSL/certs/cluee/cluee.pem;
    ssl_certificate_key D:/App/SSL/certs/cluee/private.key;

    location /mqtt {
        proxy_pass http://emqx-wss.cluee.tech:8083/mqtt;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 600s;
        proxy_set_header Host $host;
    }
}
```

### WSS验证
``` bash
<script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
 
<script>
    // 将在全局初始化一个 mqtt 变量
    //console.log(mqtt)
    // 连接选项
const options = {
      connectTimeout: 4000, // 超时时间
      // 认证信息
      clientId: 'emqx-connect-via-webstest',
      //username: '',
      //password: '',
}

//const client = mqtt.connect('ws://cluee.tech:8083/mqtt', options)
const client = mqtt.connect('wss://cluee.tech:18084/mqtt', options)

client.on('connect', (e) => {
    console.log('成功连接服务器')
     
    // 订阅一个主题
    client.subscribe('hello', { qos: 1 }, (error) => {
        if (!error) {
            console.log('订阅成功');
            client.publish('hello', 'Hello EMQ', { qos: 1, rein: false }, (error) => {
                console.log(error || '发布成功')
            })
        }
    })
})
 
</script>
```

