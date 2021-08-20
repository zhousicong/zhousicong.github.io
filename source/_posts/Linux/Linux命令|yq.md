---
title: Linux命令|yq
description: yaml命令行解析工具
categories:
  - Linux
tags:
  - TODO
abbrlink: e6efe92f
date: 2021-08-20 14:44:29
---

## 介绍
yq是一个yaml的命令行解析工具。他有点像jq处理json一样。
> 当前版本为4.x,4版本和3版本相差很大，请注意语法。

```
Usage:
  yq [flags]
  yq [command]

Available Commands:
  eval             Apply the expression to each document in each yaml file in sequence
  eval-all         Loads _all_ yaml documents of _all_ yaml files and runs expression once
  help             Help about any command
  shell-completion Generate completion script

Flags:
  -C, --colors                force print with colors
  -e, --exit-status           set exit status if there are no matches or null or false is returned
  -f, --front-matter string   (extract|process) first input as yaml front-matter. Extract will pull out the yaml content, process will run the expression against the yaml content, leaving the remaining data intact
  -h, --help                  help for yq
  -I, --indent int            sets indent level for output (default 2)
  -i, --inplace               update the yaml file inplace of first yaml file given.
  -M, --no-colors             force print with no colors
  -N, --no-doc                Don't print document separators (---)
  -n, --null-input            Don't read input, simply evaluate the expression given. Useful for creating yaml docs from scratch.
  -P, --prettyPrint           pretty print, shorthand for '... style = ""'
  -j, --tojson                output as json. Set indent to 0 to print json in one line.
      --unwrapScalar          unwrap scalar, print the value with no quotes, colors or comments (default true)
  -v, --verbose               verbose mode
  -V, --version               Print version information and quit

Use "yq [command] --help" for more information about a command.
```

**//TODO 补充实例**