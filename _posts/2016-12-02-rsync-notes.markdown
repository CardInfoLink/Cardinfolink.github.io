---
layout: post
title:  "a little of rsync notes"
date:   2016-12-02 15:14:54 +0800
categories: note
tags: rsync
author: hongmin
---

记述使用rsync daemon模式下的一些经验
### RSYNC 指南
请参考
```
man rsync
man rsyncd.conf
```
### rsync 安装
通过包管理安装
```
# for debian's
apt-get install rsync
# for redhat's
yum install rsync
```
### rsync 配置
默认配置文件/etc/rsyncd.conf，没有请创建
下面是一个简单的配置
```
port = 32456

[photo]
path = /data/photo
comment = something funn..
read only = true
auth users = rsync1,rsync2
secrets file = /etc/rsyncd.secrets
```
密码文件通常配置为/etc/rsyncd.secrets，格式很简单，需要注意的是密码一般不要超过8位
```
rsync1:11233
rsync2:3=-2234
rsync3:21134
```
### rsync启动
```
rsync --daemon
```
### rsync使用
- 列出主机上对应的rsync资源
```
rsync rsync://127.0.0.1:32456/
```
- 查看某个rsync资源的内部明细，需要输入密码
```
rsync rsync://rsync1@127.0.0.1:32456/photo
```
如果不想输入密码，可以使用环境变量指定密码
```
RSYNC_PASSWORD=11233 rsync rsync://rsync1@127.0.0.1:32456/photo
```

