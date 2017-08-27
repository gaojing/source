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

## 6.1.2 强指针sp

