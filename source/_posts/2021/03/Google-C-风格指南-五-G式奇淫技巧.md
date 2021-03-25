---
title: Google C++风格指南(五)——G式奇淫技巧
date: 2021-03-25 21:01:22
tags:
- C++编码规范
categories:
- C++
---

# 所有权与智能指针
C++11后时代C++程序员的常识题，不要使用`std::auto_ptr`，使用`std::unique_ptr`或`std::shared_ptr`，倾向于前者。

# Cpplint
风格检查[cpplint.py](https://github.com/google/styleguide/blob/gh-pages/cpplint/cpplint.py)