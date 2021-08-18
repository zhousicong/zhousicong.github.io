---
title: for循环中i++和++i的区别和效率
abbrlink: 2e389ba
categories:
  - JS
date: 2021-08-18 17:05:54
description:
tags:
---
# 背景
在看到一些书籍和视频中发现for循环里面有写++i,而不是i++
<!-- more -->

# 原理
for(A,B,C){
  D
}
执行顺序:
1. 进入循环执行A
2. 执行B
3. 执行D
4. 执行C
5. 再B，再D，再C直至退出循环

即可知上述循环等同于:
for(A,B,){
  D
  C
}


# 区别
i++
```js
// 相当于
i += 1
return i
```

++i
```js
// 相当于
j = i
i += 1
return j
```

从上述差别中可以看到是有效率上面的差别