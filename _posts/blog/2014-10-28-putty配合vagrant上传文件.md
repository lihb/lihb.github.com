---
layout: post
title: putty配合vagrant上传文件 
description: 使用putty共享文件
category: blog
---

## vagrant文件共享（vagrant独有）

在使用vagrant创建虚拟机的时候，会创建一个工作目录，这个目录就是真机和虚拟机的共享目录，这个目录在虚拟机中的地址是：**/vagrant**，将需要共享的文件放到这个目录即可完成共享。

## 利用putty工具实现真机与虚拟机的文件共享（通用）

主要用的是一个putty工具集中的一个工具软件叫**pscp.exe**

使用方法如下：

```
F:\>pscp -P 2222 PUTTY.CHM vagrant@127.0.0.1:/tmp/ # 将windows系统F盘目录下的PUTTY.CHM文件上传到vagrant虚拟机的/tmp目录下

F:\>pscp -P 2222 vagrant@127.0.0.1:/tmp/* d:\  # 将vagrant系统/tmp目录下的所有文件上传到windows系统的d盘
```

## 参考文献

1.pscp命令的帮助文档: `pscp --help`

2.[Windows下SSH客服端PuTTY–详细使用教程](http://my.oschina.net/acejimmyster/blog/26249)
