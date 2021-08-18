---
title: argo-combined-demo
categories:
  - Argo
tags:
  - Argo
abbrlink: 612a1ead
date: 2021-08-17 15:43:43
---
> Automation of Every - How To combine Argo Events, Workflow & CD

<!-- more -->

- [部署argo workflow](#部署argo-workflow)
- [部署argo events](#部署argo-events)
  - [Create the ns](#create-the-ns)
  - [Deploy Argo Events, SA, ClusterRoles, Sensor Controller, EventBus Controller and EventSource Controller](#deploy-argo-events-sa-clusterroles-sensor-controller-eventbus-controller-and-eventsource-controller)
  - [Deploy the eventbus](#deploy-the-eventbus)
  - [Create ServiceAccount to trigger workflows](#create-serviceaccount-to-trigger-workflows)
  - [Create the webhook event source](#create-the-webhook-event-source)
  - [Create the webhook sensor](#create-the-webhook-sensor)
- [部署argo cd](#部署argo-cd)

## 部署argo workflow
> 云原生工作流引擎，专注于任务编排
```shell
# 在argo-events namespace中创建argo workflow服务
kubectl create ns argo-events
# 下载argo-workflow资源文件
# 修改资源文件中RoleBinding和ClusterRoleBinding的namespace argo为argo-events
wget https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml
kubectl apply -n argo-events -f quick-start-postgres.yaml
```
## 部署argo events
> 云原生的事件驱动架构
> 部署argo events之前确保workflow已经可以工作
### Create the ns
```shell
# Create the ns
kubectl create namespace argo-events
```
### Deploy Argo Events, SA, ClusterRoles, Sensor Controller, EventBus Controller and EventSource Controller
``` shell
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install.yaml
# Install with a validating admission controller
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install-validating-webhook.yaml
```
### Deploy the eventbus
```shell
kubectl apply -n argo-events -f https://raw.githubusercontent.com/argoproj/argo-events/stable/examples/eventbus/native.yaml
```
### Create ServiceAccount to trigger workflows
<details>
<summary>service-account.yaml</summary>

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

### Create the webhook event source
> 官方不建议直接使用spec.service暴露服务，这个只是为了测试使用
<details>
<summary>event-source-example.yaml</summary>

```yaml
apiVersion: argoproj.io/v1alpha1
kind: EventSource
metadata:
  name: webhook
spec:
  service:
    ports:
      - port: 12000
        targetPort: 12000
  webhook:
    # event-source can run multiple HTTP servers. Simply define a unique port to start a new HTTP server
    example:
      # port to run HTTP server on
      port: "12000"
      # endpoint to listen to
      endpoint: /example
      # HTTP request method to allow. In this case, only POST requests are accepted
      method: POST
```
</details>
<details>
<summary>event-source.yaml & event-source-svc.yaml</summary>

```yaml
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

### Create the webhook sensor
> senor可以定义一系列的triggers,例如k8s资源对象(eg:workflow对象本身) [sensor-api](https://github.com/argoproj/argo-events/blob/master/api/sensor.md)
> 示例2可以将资源文件与代码一起提交至代码库,方便进行回归测试,也可以查看不同时期workflow的差异
<details>
<summary>示例1: 完整的workflow对象</summary>

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
            resource:
              apiVersion: argoproj.io/v1alpha1
              kind: Workflow
              metadata:
                generateName: gitlab-workflow-
              spec:
                entrypoint: argo-ci
                arguments:
                  parameters:
                    - name: repo
                      value: http://172.16.20.150/root/vue_todolist.git
                    - name: revision
                      value: main
                templates:
                  - name: argo-ci
                    steps:
                      - - name: checkout
                          template: checkout
                          arguments:
                            parameters:
                              - name: repo
                                value: "{{workflow.parameters.repo}}"
                              - name: revision
                                value: "{{workflow.parameters.revision}}"
                      - - name: build
                          template: build
                          arguments:
                            artifacts:
                              - name: source
                                from: "{{steps.checkout.outputs.artifacts.source}}"
                  - name: checkout
                    inputs:
                      parameters:
                        - name: repo
                        - name: revision
                      artifacts:
                        - name: source
                          path: /src
                          git:
                            repo: "{{inputs.parameters.repo}}"
                            revision: "{{inputs.parameters.revision}}"
                    outputs:
                      artifacts:
                        - name: source
                          path: /src
                    container:
                      image: my-ubuntu:v0.1
                      command: [sh, -c]
                      args: ["cd /src && git status && ls -l"]
                  - name: build
                    inputs:
                      artifacts:
                        - name: source
                          path: /src
                    outputs:
                      artifacts:
                        - name: source
                          path: /src
                    container:
                      image: my-ubuntu:v0.1
                      command: [sh, -c]
                      args: [
                          "
                          cd /src &&
                          npm install --registry https://registry.npm.taobao.org &&
                          npm run build
                          ",
                        ]
          parameters:
            - src:
                dependencyName: test-dep
                dataKey: body.project.git_http_url
              dest: spec.arguments.parameters.0.value
            - src:
                dependencyName: test-dep
                dataKey: body.ref
              dest: spec.arguments.parameters.1.value

```
</details>

<details>
<summary>示例2: 访问git仓库中的资源文件</summary>

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

## 部署argo cd
```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```