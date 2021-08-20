---
title: Argo Workflow实践|安装部署
description: Kubernetes 原生工作流
categories:
- Argo
- Argo Workflow
abbrlink: 1e633edc
date: 2021-08-19 14:44:13
tags:
- Argo
---
## 简介
　　Argo Workflow是一个云原生的工作流引擎，专注于编排并行任务。特点如下:
- 使用K8S的CRD来定义工作流，本身也是CRD，可以使用yml直接定义
- 工作流的每一步都是一个容器
- 可以将多步骤工作流建模为一些列的任务，或者使用DAG(有向无环图)描述任务之间的关系
- 可以短时间内轻松运行用于机器学习或数据处理的计算密集型作业

## 安装
> 请先安装K8S,本例采用快捷方式安装
> 本文安装时间为202108018,如果采用官网文档推荐[资源文件](https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml)会出现浏览器访问报错但是命令行访问argo server正常的情况,[相关issue](https://github.com/argoproj/argo-workflows/issues/5573)
> 本例直接使用3.1.6版本资源文件，此版本已fix上述issue

```
kubectl create ns argo
kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/download/v3.1.6/quick-start-postgres.yaml
```
安装完成后，会有如下4个pod:
```
NAME                                  READY   STATUS    RESTARTS   AGE
argo-server-7949b545cc-szs9h          1/1     Running   0          3m49s
minio-77d6796f8d-g6rsv                1/1     Running   0          3m49s
postgres-546d9d68b-4xtrq              1/1     Running   0          3m49s
workflow-controller-bd8bc5f86-kxsfv   1/1     Running   0          3m49s
```
其中:
- argo-server为argo服务端
- minio为制品仓库
- postgres为数据库
- workflow-controller为流程控制器

## 访问Argo Workflows UI
1. kubectl port-forward
```
kubectl -n argo port-forward svc/argo-server 2746:2746
```
2. Expose a loadBalancer
```
kubectl patch svc argo-server -n argo -p '{"spec": {"type": "LoadBalancer"}}'

NAME                          TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
argo-server                   LoadBalancer   10.110.3.94     <pending>     2746:31603/TCP   12m
minio                         ClusterIP      10.111.108.35   <none>        9000/TCP         12m
postgres                      ClusterIP      10.97.250.221   <none>        5432/TCP         12m
workflow-controller-metrics   ClusterIP      10.107.148.72   <none>        9090/TCP         12m
```
3. Ingress
`示例略`

![Argo Workflow UI](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108191531513.png)

## 示例验证
```
> argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml

apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world-
  labels:
    workflows.argoproj.io/archive-strategy: "false"
  annotations:
    workflows.argoproj.io/description: |
      This is a simple hello world example.
      You can also run it in Python: https://couler-proj.github.io/couler/examples/#hello-world
spec:
  entrypoint: whalesay
  templates:
  - name: whalesay
    container:
      image: docker/whalesay:latest
      command: [cowsay]
      args: ["hello world"]

> argo list -n argo
NAME                STATUS      AGE   DURATION   PRIORITY
hello-world-hphdr   Succeeded   43s   10s        0


> argo get -n argo @latest
Name:                hello-world-hphdr
Namespace:           argo-events
ServiceAccount:      default
Status:              Succeeded
Conditions:
 PodRunning          False
 Completed           True
Created:             Thu Aug 19 15:34:37 +0800 (1 minute ago)
Started:             Thu Aug 19 15:34:37 +0800 (1 minute ago)
Finished:            Thu Aug 19 15:34:47 +0800 (1 minute ago)
Duration:            10 seconds
Progress:            1/1
ResourcesDuration:   7s*(1 cpu),7s*(100Mi memory)

STEP                  TEMPLATE  PODNAME            DURATION  MESSAGE
 ✔ hello-world-hphdr  whalesay  hello-world-hphdr  7s

This workflow does not have security context set. You can run your workflow pods more securely by setting it.
Learn more at https://argoproj.github.io/argo-workflows/workflow-pod-security-context/

> argo logs -n argo @latest
hello-world-hphdr:  _____________
hello-world-hphdr: < hello world >
hello-world-hphdr:  -------------
hello-world-hphdr:     \
hello-world-hphdr:      \
hello-world-hphdr:       \
hello-world-hphdr:                     ##        .
hello-world-hphdr:               ## ## ##       ==
hello-world-hphdr:            ## ## ## ##      ===
hello-world-hphdr:        /""""""""""""""""___/ ===
hello-world-hphdr:   ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
hello-world-hphdr:        \______ o          __/
hello-world-hphdr:         \    \        __/
hello-world-hphdr:           \____\______/
```
![hello-workd-wf](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108191537622.png)