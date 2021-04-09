---
title: assert的那些事
date: 2021-04-09 19:44:43
tags:
- assert
categories:
- c++
---

# 问题描述
今天跟算法同学沟通需求，需要对之前写过的一个DS的接口进行改动，在改代码的过程中，突然发现DS初始化里的`assert(!_initialized)`检查写成了`assert(_initialized)`。

理论上，在进行该DS初始化时应当初始化失败，程序停止并报错，但之前测试和调用都是正常的。

因为之前写python写的比较多，对C++的各种细节理解并不深刻，只会写一些简单的逻辑，因此并不清楚assert相关的细节。

# 问题分析
查看`assert.h`头文件
```c++

```
