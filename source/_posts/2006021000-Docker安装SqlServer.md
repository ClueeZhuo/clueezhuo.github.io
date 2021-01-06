---
title: 2006021000@Docker安装SqlServer
date: 2020-06-02 09:57:42
categories:
- SQL
tags:
- MSSQL
---
# 下载镜像
https://hub.docker.com/_/microsoft-mssql-server

# 运行镜像并挂载宿主机目录

```shell
mkdir /var/lib/docker/volumes/mssql_data

chmod 777 /var/lib/docker/volumes/mssql_data

sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=XXX" \
   -p 51433:1433 --name mssql \
   -v /var/lib/docker/volumes/mssql_data:/var/opt/mssql \
   -d mcr.microsoft.com/mssql/server:2019-CU3-ubuntu-18.04
```