---
title: Docker部署gitlab
categories:
- 博客
abbrlink: e32bdacf
---
> Mac环境本地搭建Gitlab进行测试

<!-- more -->

- [1.环境准备](#1环境准备)
  - [1.1 拉取最新容器](#11-拉取最新容器)
  - [1.2 构建容器](#12-构建容器)
    - [1.2.1 创建目录，用于数据持久化](#121-创建目录用于数据持久化)
    - [1.2.2 构建容器](#122-构建容器)
    - [1.2.3 查看容器](#123-查看容器)
- [2. 配置Gitlab访问地址](#2-配置gitlab访问地址)
- [3. 访问服务](#3-访问服务)
- [4. mac访问宿主机网络](#4-mac访问宿主机网络)

## 1.环境准备
### 1.1 拉取最新容器
```
docker pull gitlab/gitlab-ce:latest
``` 
### 1.2 构建容器
#### 1.2.1 创建目录，用于数据持久化
```
mkdir -p ${gitlab-home}/{data,config,logs}
```
#### 1.2.2 构建容器
```
docker run -dit \
-p 443:443 -p 80:80 -p 222:22 \
--name gitlab --restart always \
-v ${gitlab-home}/config:/etc/gitlab \
-v ${gitlab-home}/logs:/var/log/gitlab \
-v ${gitlab-home}/data:/var/opt/gitlab \
gitlab/gitlab-ce
```
#### 1.2.3 查看容器
```
docker ps -a | grep gitlab
```
## 2. 配置Gitlab访问地址
```
# step1: 进入容器
# step2: vim /etc/gitlab/gitlab.rb 修改 external_url
# step3: 重启容器
```
## 3. 访问服务
```
## 管理员默认用户 root
## 管理员默认密码 /etc/gitlab/initial_root_password
```
## 4. mac访问宿主机网络
```
# step1: 使用Gitlab管理员账号修改如下配置
Menu => Admin => Settings => Network => Outbound requests
勾选Allow requests to the local network from web hooks and services
# step2: container中使用docker.for.mac.localhost访问宿主机
```