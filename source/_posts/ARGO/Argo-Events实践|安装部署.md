---
title: Argo Events实战|安装部署
description: Kubernetes的事件驱动的工作流自动化框架
categories:
- Argo
- Argo Events
abbrlink: ae06cf7a
date: 2021-08-19 17:54:37
tags:
- Argo
---
## 简介
Argo Events是Kubernetes的事件驱动的工作流自动化框架.

## 组件
- Event Source
  > 类似Gateway,把消息发送给eventbus
- Sensor
  > 时间参数化并对时间过滤
- EventBus
  > 时间消息队列
- Trigger
  > k8S CRD


## 安装
> 确保已经安装Argo Workflow
### 1. 创建命名空间
```shell
kubectl create ns argo-events
```
### 2. 部署argo events相关组件
```shell
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install.yaml
## Install with a validating admission controller
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install-validating-webhook.yaml

```
### 3. 部署eventbus
```shell
kubectl apply -n argo-events -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/eventbus/native.yaml
```
### 4. 创建能够触发Workflow的用户
<details>
<summary>user.yaml</summary>

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: argo-events
  name: operate-workflow-sa
---
# Similarly you can use a ClusterRole and ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: operate-workflow-role
  namespace: argo-events
rules:
  - apiGroups:
      - argoproj.io
    verbs:
      - "*"
    resources:
      - workflows
      - workflowtemplates
      - cronworkflows
      - clusterworkflowtemplates
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: operate-workflow-role-binding
  namespace: argo-events
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: operate-workflow-role
subjects:
  - kind: ServiceAccount
    name: operate-workflow-sa
```
</details>

### 5. 部署event source
> 用于接收请求

<details>
<summary>eventsource.yaml</summary>

```
apiVersion: argoproj.io/v1alpha1
kind: EventSource
metadata:
  name: gitlab-eventsource
spec:
  webhook:
    gitlab-example:
      port: "13000"
      endpoint: /webhook
      method: POST
---
apiVersion: v1
kind: Service
metadata:
  name: gitlab-webhook
spec:
  selector:
    eventsource-name: gitlab-eventsource
  ports:
    - port: 13000
      targetPort: 13000
  type: LoadBalancer
```
</details>

### 6. 部署Sensor服务
> 消费请求

<details>
<summary>sensor.yaml</summary>

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Sensor
metadata:
  name: gitlab
spec:
  template:
    serviceAccountName: operate-workflow-sa
  dependencies:
    - name: test-dep
      eventSourceName: gitlab-eventsource
      eventName: gitlab-example
  triggers:
    - template:
        name: gitlab-workflow-trigger
        k8s:
          group: argoproj.io
          version: v1alpha1
          resource: workflows
          operation: create
          source:
            git:
              filePath: "workflow/gitlab.yaml"
          parameters:
            - src:
                dependencyName: test-dep
                dataKey: body.project.git_http_url
              dest: spec.arguments.parameters.0.value
            - src:
                dependencyName: test-dep
                dataKey: body.ref
              dest: spec.arguments.parameters.1.value
      parameters:
        - src:
            dependencyName: test-dep
            dataKey: body.project.git_http_url
          dest: k8s.source.git.url
        - src:
            dependencyName: test-dep
            dataKey: body.ref
          dest: k8s.source.git.ref
```
</details>

### 7. 验证
```
❯ curl -d '{"message":"this is my first webhook"}' -H "Content-Type: application/json" -X POST http://localhost:13000/webhook
success
```