---
title: K8S中port|nodePort|targetPort概念的区分
categories:
  - K8S
abbrlink: 68b4a711
date: 2021-08-20 09:46:24
description:
tags:
---
## 概念
- port是service的端口
- targetPort是容器的端口
- nodePort是容器所在宿主机的端口

<!-- more -->

## 作用
### port
集群内部服务暴露的端口，可以使用clusterIp:port来访问pod暴露的服务

### targetPort
最终容器上面暴露的服务端口，应用监听的端口

### nodePort
当服务类型为NodePort时，会在每个节点上暴露一个端口,可以他通过nodeIp:nodePort从集群外部访问服务。
nodeport默认端口范围为(30000-32767)

## 总结
nodePort集群外部访问,pod为集群内部pod相互通信端口,targetPort字面意思就是容器暴露端口。