title: 深入理解Android内核设计思想 第六章 进程间通信 binder
categories: 深入理解Android内核设计思想
tags: [深入理解Android内核设计思想]
---
Binder
- Binder驱动
- Service Manager
- Binder Client
- Binder Server

Binder的各个组成元素和TCP/IP网络相似
- Binder驱动  -- 路由器
- Service Manager -- DNS
- Binder Client -- 客户端
- Binder Server -- 服务器

## 6.1 智能指针

### 6.1.3 弱指针
主要解决两个对象相互引用导致无法释放的情况。

## 6.3 Binder驱动与协议
Binder驱动会将自己注册成一个misc device，并向上层提供一个/dev/binder节点。

### 6.3.1 打开binder驱动-binder_open
上层进程访问binder驱动时，先要打开/dev/binder节点，体现在binder_open中

### 6.3.2 binder_mmap
上层用户调用mmap系统调用时最终调用binder_mmap。

