---
title: CEPH客户端版本升级
description: 修改默认Ubuntu软件源，升级ceph客户端版本至N版本(14.x.x)
categories:
  - CEPH
abbrlink: bab85c8a
date: 2021-08-17 15:19:48
tags:
---
```
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
echo deb https://download.ceph.com/debian-nautilus/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
apt update
apt install ceph-common
```