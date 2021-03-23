---
title: C++宏定义的那些事
date: 2021-03-23 21:57:57
tags: 
- C++ 
categories:
- C++相关
---


# 简单的宏
```c++
//第一种
#define PI2 3.1415926
#define Add(x) x+1 //错误示范
#define Add(x) (x+1)

//第二种
#define TEST
#ifdef TEST
std::cout << "this is a TEST " << endl;
#ifndef TEST
std::cout << "there are not TEST" << endl;
#endif
```
宏的两种简单用法如上所示，第一种用法将`PI`全部替换为`3.1415926`，这里的替换是直接替换，错误示范
