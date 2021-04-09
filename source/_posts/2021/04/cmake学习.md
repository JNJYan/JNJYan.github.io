---
title: cmake学习
date: 2021-04-09 15:31:38
tags:
categories:
---

# 简介
CMake是一个编译系统生成工具，而非编译系统。CMake能够生成编译系统的输入文件如`Makefile`，CMake本身支持`Make/Ninja/Visual Studio/XCode`等。

CMake是跨平台的，支持Linux、Windows、OSX等，同时也支持跨平台构建（编译器要支持跨平台才可以哦）。

CMake开始于1999/2000年，现代CMake开始于2014年的`3.0`版本，现代CMake有一个非常重要的概念，`Everything is a (self-contained) target`。

> Everything that is needed to (successfully) build that target.
>  
> Everything that is needed to (successfully) use that target.

# 起手式
- Declare a target
- Declare target's traits
- It's all about targets

```cmake
cmake_minimum_required(VERSION 3.5)

project(hello-world)

# 静态库
add_library(hello hello.cc)
# 动态库
add_library(hello SHARED hello.cc)

add_subdirectory(src)

install(FILES COPYRIGHT README.md DESTINATION share/doc/cmake/hello-world)
install(PROGREAMS **.sh DESTINATION bin)
install(DIRECTORY doc DESTINATION share/doc/cmake/hello-world)
```

# 参考文献
- [modern-cmake](https://cliutils.gitlab.io/modern-cmake/)
- [youtube/c++2018](https://www.youtube.com/watch?v=y7ndUhdQuU8)