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

### spawn
spawn 方法会在新的流程智行外部应用，返回I/O的一个流接口。
```js
const {spawn} = require('child_process')
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});

const ps = spawn('ps', ['ax']);
const grep = spawn('grep', ['ssh']);

ps.stdout.on('data', (data) => {
  grep.stdin.write(data);
});

ps.stderr.on('data', (data) => {
  console.error(`ps stderr: ${data}`);
});

ps.on('close', (code) => {
  if (code !== 0) {
    console.log(`ps process exited with code ${code}`);
  }
  grep.stdin.end();
});

grep.stdout.on('data', (data) => {
  console.log(data.toString());
});

grep.stderr.on('data', (data) => {
  console.error(`grep stderr: ${data}`);
});

grep.on('close', (code) => {
  if (code !== 0) {
    console.log(`grep process exited with code ${code}`);
  }
});
```

### exec
exec 方法会在衍生的shell中执行command,缓冲任何生成的输出。
`exec只有利用shell功能才能使用`
```js
const { exec } = require('child_process');
exec('cat *.js missing_file | wc -l', (error, stdout, stderr) => {
  if (error) {
    console.error(`exec error: ${error}`);
    return;
  }
  console.log(`stdout: ${stdout}`);
  console.error(`stderr: ${stderr}`);
});
```

### execFile
execFile 方法他不会默认衍生shell,而是指定可执行文件直接作为新进程衍生。但是由于未衍生shell，因此不支持I/O重定向和文件通配等行为。
```js
const { execFile } = require('child_process');
const child = execFile('node', ['--version'], (error, stdout, stderr) => {
  if (error) {
    throw error;
  }
  console.log(stdout);
});
```

### fork
fork 方法是spawn的特殊情况，专门用于衍生新的nodejs进程，返回一个child_process对象。