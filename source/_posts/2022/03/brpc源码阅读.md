---
title: brpc源码阅读——bvar篇
date: 2022-03-12 14:45:17
tags:
categories:
---

bvar是多线程环境下的计数器类库，利用thread local存储减少了cache bouncing