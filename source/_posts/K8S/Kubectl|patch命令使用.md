---
title: kubectl|patch命令使用
description: '使用patch(补丁)修改、更新资源的字段,支持JSON和YAML格式。'
categories:
  - K8S
abbrlink: e6afd30a
date: 2021-08-20 09:34:05
tags:
- kubectl命令
- TODO
---

## 语法
`kubectl patch (-f FILENAME | TYPE NAME) -p PATCH`

## 示例
```
# Partially update a node using a strategic merge patch. Specify the patch as JSON.
kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}'

# Partially update a node using a strategic merge patch. Specify the patch as YAML.
kubectl patch node k8s-node-1 -p $'spec:\n unschedulable: true'

# Partially update a node identified by the type and name specified in "node.json" using strategic mergepatch.
kubectl patch -f node.json -p '{"spec":{"unschedulable":true}}'

# Update a container's image; spec.containers[*].name is required because it's a merge key.
kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"newimage"}]}}'

# Update a container's image using a json patch with positional arrays.
kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path": "/spec/containers/0/image","value":"newimage"}]'
```

## 参数
```
-f, --filename=[]: Filename, directory, or URL to a file identifying the resource to update
    --include-extended-apis[=true]: If true, include definitions of new APIs via calls to the APIserver. [default true]
-o, --output="": Output mode. Use "-o name" for shorter output (resource/name).
-p, --patch="": The patch to be applied to the resource JSON file.
    --record[=false]: Record current kubectl command in the resource annotation.
-R, --recursive[=false]: Process the directory used in -f, --filename recursively. Useful when you want to manage related manifests organized within the same directory.
    --type="strategic": The type of patch being provided; one of [json merge strategic]
```

**//TODO 增加一些实战示例**