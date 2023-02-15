---
title: Netty
toc: content
keywords: [middle]
---

## 简介

- 在NIO上做了很多优化
  - 零拷贝
  - 高性能无锁队列
  - 内存池
- 支持多种通信协议
  - http
  - websocket
- 解决==拆包粘包==的问题，内置了拆包策略