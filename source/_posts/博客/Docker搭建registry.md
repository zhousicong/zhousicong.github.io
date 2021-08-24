---
title: Docker搭建registry
abbrlink: 2adf5bb7
date: 2021-08-24 16:13:27
description:
categories:
tags:
---
　　工作中当我们执行docker pull时，它实际上是从registry.hub.docker.com上面去查找，这是官方提供的公共仓库。实际项目中，我们不可能将企业私有镜像push到公共仓库。所以为了更好的管理镜像,docke允许我们搭建本地私有仓库-registry。
　　registry使用比较简单，但是管理功能较差。企业级别的私服推荐还是使用harbor提供服务。

<!-- more -->

## 私有仓库搭建
### 镜像拉取
```
docker pull registry:2
```

### 创建并启动仓库容器
```
docker run -itd --name=registry -p 5000:5000 registry:2
```

### 访问私有仓库
1. 命令行访问
```
❯ curl localhost:5000/v2/_catalog
{"repositories":[]}
```
2. 浏览器访问 http://localhost:5000/v2/_catalog
![registry-1](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108241628420.png)

### 修改daemon.json(让docker信任私有仓库地址)
`如果没有修改daemon.json,可以成功上传镜像，但是无法下载镜像`
```
# vim daemon.json
# 添加一下内容
"insecure-registries":["localhost:5000"]
```

### 上传镜像至私有仓库
```
❯ docker images | grep nginx
nginx                                  latest                                                  dd34e67e3371   6 days ago      133MB

❯ docker tag dd34e67e3371 localhost:5000/nginx:latest

❯ docker images | grep nginx
nginx                                  latest                                                  dd34e67e3371   6 days ago      133MB
localhost:5000/nginx                   latest                                                  dd34e67e3371   6 days ago      133MB

❯ docker push localhost:5000/nginx:latest
The push refers to repository [localhost:5000/nginx]
fb04ab8effa8: Pushed
8f736d52032f: Pushed
009f1d338b57: Pushed
678bbd796838: Pushed
d1279c519351: Pushed
f68ef921efae: Pushed
latest: digest: sha256:5e95e5eb8be4322e3b3652d737371705e56809ed8b307ad68ec59ddebaaf60e4 size: 1570
```

### 访问私有仓库-again
1. 命令行访问
```
❯ curl localhost:5000/v2/_catalog
{"repositories":["nginx"]}
```
2. 浏览器访问 http://localhost:5000/v2/_catalog
![registry-2](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108241635105.png)