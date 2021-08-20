---
title: K8S|Secret详解
abbrlink: 5a8a6c8d
date: 2021-08-20 15:26:42
description: Secret概述与类型说明，并详解常用Secret示例
categories:
- K8S
- Secret
tags:
- K8S
---

## 介绍
Secret解决了密码、token、秘钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者Pod Spec中。Secret可以以Volume或者环境变量的方式使用。

## Secret类型
- Service Account：用来访问Kubernetes API，由Kubernetes自动创建，并且会自动挂载到Pod的 /run/secrets/kubernetes.io/serviceaccount 目录中。
- Opaque：base64编码格式的Secret，用来存储密码、秘钥等。
- kubernetes.io/dockerconfigjson：用来存储私有docker registry的认证信息。
> 本文主要使用Opaque

```shell
# 命令行直接创建
kubectl create secret generic my-secret --from-literal=username=root --from-literal=password=root

# 使用文件创建
echo -n "root" > ./username
echo -n "root" > ./password
kubectl create secret generic my-secret --from-file=./username --from-file=./password

# 先对字符串进行base64加密
echo -n 'root' | base64
cm9vdA==

# 编辑资源文件
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: base64
  password: base64

# 解密
kubectl get secrets my-secret -o jsonpath="{.data.username}" | base64 -d
root
```

