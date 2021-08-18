---
title: Linux命令|cut
categories:
  - Linux命令
abbrlink: 5b806bba
date: 2021-08-18 16:53:14
description:
tags:
---
# cut
# 参数
```
-b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。
-c ：以字符为单位进行分割。
-d ：自定义分隔符，默认为制表符。
-f ：与-d一起使用，指定显示哪个区域。
-n ：取消分割多字节字符。仅和 -b 标志一起使用。如果字符的最后一个字节落在由 -b 标志的 List 参数指示的范围之内，该字符将被写出；否则，该字符将被排除
```
# 示例
```shell
ping www.baidu.com  -c1 | grep PING | cut -d '(' -f2| cut -d ')' -f1
```