---
title: 2104091338@CentOS7配置Supervisor开机启动
categories:
  - CentOS
tags:
  - Supervisor
date: 2021-04-09 13:38:56
---

### 1. 在自己桌面新建一个supervisord.service文件

内容为: 
```shell
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/local/bin/supervisord -c /etc/supervisor/supervisord.conf
ExecStop=/usr/local/bin/supervisorctl shutdown
ExecReload=/usr/local/bin/supervisorctl reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

### 2. 将此文件拷贝到/usr/lib/systemd/system文件夹下

### 3. 执行命令:

```shell
systemctl enable supervisord
```
 

### 4. 执行命令测试是否为开机启动

```shell
systemctl is-enabled supervisord
```