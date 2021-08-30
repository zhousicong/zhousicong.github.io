---
title: Golang开发环境配置
description: 开发环境搭建
categories:
  - GO
tags:
  - GO
abbrlink: b66fdee6
date: 2021-08-26 15:59:13
---

## 开发环境搭建
---
### 安装Golang
1. Golang[官网](https://golang.org/),进入下载页面，选择对应的版本进行下载安装
2. mac可以通过Homebrew `brew install go`进行安装

>安装完成后可以通过运行`go env`查看是否安装成功

### 关于GOROOT和GOPATH
#### GOROOT
GOROOT就是你go的安装目录,使用Homebrew安装对应的目录就是<font color="DeepPink">/usr/local/Cellar/go/1.16.5/libexec</font>。
#### GOPATH
GOPATH是go的工作目录。谈GOPATH时需要引入一个环境变量`GO111MODULE`。
写本文时的go安装版本为1.16.5。GO111MODULE的参数为off、on和auto,其不同值的表现行为如下:
- auto (默认值)
  - 当项目路径(go.mod)在GOPATH外时，其行为等同于GO111MODULE=on。这意味着可以将代码仓库存储于GOPATH之外
  - 当项目路径在GOPATH内，并且没有go.mod文件存在时，其行为等同于GO111MODULE=off
- off
无模块支持，go会从GOPATH和vender文件夹寻找包
- on
模块支持,go会忽略GOPATH和vender文件夹，只根据<font color="DeepPink">go.mod</font>下载依赖。