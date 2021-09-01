---
title: NODEJS|child_process模块
description: child_process子进程
categories:
  - NODEJS
tags:
  - NODEJS
abbrlink: 1375a0ac
date: 2021-08-30 17:33:27
---

child_process.spawn() 方法异步衍生子进程，不会阻塞 Node.js 事件循环。 child_process.spawnSync() 函数以同步方式提供等效的功能，其会阻塞事件循环，直到衍生的进程退出或终止。

为方便起见，child_process 模块提供了一些同步和异步方法替代 child_process.spawn() 和 child_process.spawnSync()。 这些替代方法中的每一个都是基于 child_process.spawn() 或 child_process.spawnSync() 实现。

- child_process.exec(): 衍生 shell 并在该 shell 中运行命令，完成后将 stdout 和 stderr 传给回调函数。
- child_process.execFile(): 与 child_process.exec() 类似，不同之处在于，默认情况下，它直接衍生命令，而不先衍生 shell。
- child_process.fork(): 衍生新的 Node.js 进程并使用建立的 IPC 通信通道（其允许在父子进程之间发送消息）调用指定的模块。
- child_process.execSync(): child_process.exec() 的同步版本，其将阻塞 Node.js 事件循环。
- child_process.execFileSync(): child_process.execFile() 的同步版本，其将阻塞 Node.js 事件循环。
对于某些情况，例如自动化 shell 脚本，同步的方法可能更方便。 但是，在许多情况下，由于在衍生的进程完成前会停止事件循环，同步方法会对性能产生重大影响。

-- 摘自[NODEJS文档-child_process](http://nodejs.cn/api/child_process.html)