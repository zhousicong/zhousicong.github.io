---
title: Argo CD实战|安装部署
categories:
  - Argo
  - Argo CD
tags:
  - Argo
abbrlink: a25ee031
date: 2021-08-23 11:49:37
description:
---
## 介绍
　　Argo CD 是一个为 Kubernetes 而生的，遵循声明式 GitOps 理念的持续部署工具。Argo CD 可在 Git 存储库更改时自动同步和部署应用程序。

<!-- more -->

　　Argo CD 遵循 GitOps 模式，使用 Git 仓库作为定义所需应用程序状态的真实来源，Argo CD 支持多种 Kubernetes 清单：
- kustomize
- helm charts
- ksonnet applications
- jsonnet files
- Plain directory of YAML/json manifests
- Any custom config management tool configured as a config management plugin
  
　　Argo CD 可在指定的目标环境中自动部署所需的应用程序状态，应用程序部署可以在 Git 提交时跟踪对分支、标签的更新，或固定到清单的指定版本。

## 核心概念

- Application：应用，一组由资源清单定义的 Kubernetes 资源，这是一个 CRD 资源对象
- Application source type：用来构建应用的工具
- Target state：目标状态，指应用程序所需的期望状态，由 Git 存储库中的文件表示
- Live state：实时状态，指应用程序实时的状态，比如部署了哪些 Pods 等真实状态
- Sync status：同步状态表示实时状态是否与目标状态一致，部署的应用是否与 Git 所描述的一样？
- Sync：同步指将应用程序迁移到其目标状态的过程，比如通过对 Kubernetes 集群应用变更
- Sync operation status：同步操作状态指的是同步是否成功
- Refresh：刷新是指将 Git 中的最新代码与实时状态进行比较，弄清楚有什么不同
- Health：应用程序的健康状况，它是否正常运行？能否为请求提供服务？
- Tool：工具指从文件目录创建清单的工具，例如 Kustomize 或 Ksonnet 等
- Configuration management tool：配置管理工具
- Configuration management plugin：配置管理插件

## 部署
### 创建命名空间
```
kubectl create namespace argocd
```
### 创建服务
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl get po -n argocd
NAME                                  READY   STATUS    RESTARTS   AGE
argocd-application-controller-0       1/1     Running   0          3d19h
argocd-dex-server-68c7bf5fdd-n4wnw    1/1     Running   0          3d19h
argocd-redis-7547547c4f-2297t         1/1     Running   0          3d19h
argocd-repo-server-58f87478b8-zrc7f   1/1     Running   0          3d19h
argocd-server-6f4fcdc5dc-c8z54        1/1     Running   0          3d19h
```
### 暴露Argocd API Server
```
# 可以修改type为LoadBalancer
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
# 亦可以直接编辑argocd-server service
......
spec:
  clusterIP: 10.111.50.107
  clusterIPs:
  - 10.111.50.107
  externalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: http
    nodePort: 32714
    port: 8080
    protocol: TCP
    targetPort: 8080
  - name: https
    nodePort: 31315
    port: 8443
    protocol: TCP
    targetPort: 8080
  selector:
    app.kubernetes.io/name: argocd-server
  sessionAffinity: None
  type: LoadBalancer
```
### 访问Argocd
```
# 下载argocd cli
brew install argocd 
# 获取初始化密码
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
# 通过命令行登录argocd
argocd login localhost
# 命令行修改密码
argocd account update-password
```
![argocd-ui-1](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108241039178.png)

### Argocd UI创建APP
![argocd-ui-2](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108241650638.png)
![argocd-ui-3](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108241652922.png)
![argocd-ui-4](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108241653505.png)
![argocd-ui-5](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108241655840.png)
![argocd-ui-6](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108241656082.png)

### Argocd cli创建APP
```
argocd app create my-app --repo http://172.16.20.150/devops/vue_todolist.git --path deploy --dest-namespace test --dest-server https://kubernetes.default.svc --directory-recurse
```
![argocd-ui-7](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108251023860.png)

### Argocd cli同步APP
```
❯ argocd app sync my-app
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2021-08-25T10:24:17+08:00            Service        test         nginx-service  OutOfSync  Missing
2021-08-25T10:24:17+08:00   apps  Deployment        test          nginx-deploy  OutOfSync  Missing
2021-08-25T10:24:18+08:00            Service        test         nginx-service    Synced  Healthy
2021-08-25T10:24:19+08:00            Service        test         nginx-service    Synced   Healthy              service/nginx-service created
2021-08-25T10:24:19+08:00   apps  Deployment        test          nginx-deploy  OutOfSync  Missing              deployment.apps/nginx-deploy created
2021-08-25T10:24:19+08:00   apps  Deployment        test          nginx-deploy    Synced  Progressing              deployment.apps/nginx-deploy created

Name:               my-app
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          test
URL:                https://localhost:8080/applications/my-app
Repo:               http://172.16.20.150/devops/vue_todolist.git
Target:
Path:               deploy
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to  (3ad8e19)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      3ad8e19051f9bbf003800790d50ef7fd26cda7c1
Phase:              Succeeded
Start:              2021-08-25 10:24:17 +0800 CST
Finished:           2021-08-25 10:24:19 +0800 CST
Duration:           2s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME           STATUS  HEALTH       HOOK  MESSAGE
       Service     test       nginx-service  Synced  Healthy            service/nginx-service created
apps   Deployment  test       nginx-deploy   Synced  Progressing        deployment.apps/nginx-deploy created
```
![argocd-ui-7](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108251025334.png)
![argocd-ui-8](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202108251026671.png)