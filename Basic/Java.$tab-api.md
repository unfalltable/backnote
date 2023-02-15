---
title: JavaApi
keywords: [Java, advance]
---

## API

### Runtime

- 概述

  - 封装了运行的环境，每个Java应用程序都有一个Runtime实例
  - 一般不能实例化Rumtime对象，可以通过getRuntime()获取当前对象的引用
  - 拥有一个静态初始化对象--饿汉模式(单例模式的一种)
    - `private static Runtime currentRuntime = new Runtime();`

  - 常用方法

    - availableProcessors()								返回当前可用的虚拟机数量

    - exec()                                                    执行其他程序，例如打开记事本

      - `r.exec("notepad")`

    - 内存管理

      - totalMemory())						返回当前对象的堆内存有多大

      - freeMemory()                      返回当前对象的堆内存还剩多少

      - gc()                                      根据需要运行无用单元收集器先于

        ​														虚拟机自动回收对象

### Process

### Optional

- 为了防止空指针异常

#### 方法

- Optional.of(T t)
  - t不能为null
- Optional.empty()
  - 创建一个空的Optional实例
- Optional.ofNullable(T t)
  - t可以为空
- Optional对象.orElse(T t)
  - Optional对象为空则返回t
