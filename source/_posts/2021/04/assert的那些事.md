---
title: assert的那些事
date: 2021-04-09 19:44:43
tags:
- assert
categories:
- c++
---

## 问题描述

今天跟算法同学沟通需求，需要对之前写过的一个DS的接口进行改动，在改代码的过程中，突然发现DS初始化里的`assert(!_initialized)`检查写成了`assert(_initialized)`。

理论上，在进行该DS初始化时应当初始化失败，程序停止并报错，但之前测试和调用都是正常的。

因为之前写python写的比较多，对C++的各种细节理解并不深刻，因此并不清楚assert相关的细节，只当它是与python中的assert是相同的。

## 问题分析

### assert实现

```c++
// assert.h
/* void assert (int expression);

   If NDEBUG is defined, do nothing.
   If not, and EXPRESSION is zero, print an error message and abort.  */
#define __ASSERT_VOID_CAST static_cast<void>
#ifdef NDEBUG

 #define assert(expr) (__ASSERT_VOID_CAST(0))

#else
 extern void __assert_fail(char *__assertion, char *__file, uint __line, char *__function) __THROW __attribute__ ((__noreturn__));

 #define __ASSERT_FUNCTION __func__

 #define assert(expr) \
    ((expr)?__ASSERT_VOID_CAST(0):__assert_fail(#expr, __FILE__, __LINE__, __ASSERT_FUNCTION)))
#endif
```

在网上查阅资料之后发现，在VC编译器中`assert`只在DEBUG模式下生效，在RELEASE模式下不会进行编译。

查看`assert.h`头文件，头文件中的注释很清楚的写着如果定义了`NDEBUG`，那assert不会做任何事，如果未定义`NDEBUG`且表达式的值为0，就会打印错误信息并终止程序。

接下来我们来看下`assert`的实现，可以看到`assert()`使用宏来实现的，当定义了`NDEBUG`时，宏将会替换为`void(0)`，不会做任何事，若未定义`NDEBUG`，则宏会首先判断表达式`expr`是否为真，若为真则什么都不做，若为假则打印错误信息并终止程序。

`__attribute__ ((__noreturn__))`告知编译器，该函数不会返回。

`__assert_fail()`是一个二进制标准库的函数，将会终止程序，详见[__assert_fail](https://refspecs.linuxbase.org/LSB_5.0.0/LSB-Core-generic/LSB-Core-generic/baselib---assert-fail-1.html)。

### 问题追溯

在我们的编译系统，采用CMake生成Makefile，并且利用conan来管理依赖库版本，默认情况下CMake将会采用RELEASE进行编译。

## 注意事项

我们经常有如下场景，需要对某一个函数的返回值进行类型检查，若函数正常执行则返回`true`，否则返回`false`。当这个函数的正确执行与否与程序状态相关时，我们需要对其进行做如下检查:

```c++
assert(func(args));
```

由于`assert`的原理，在Release模式下，`assert(expr)`将会被宏替换为`void(0)`，因此`func(args)`的逻辑并不会执行。

```c++
bool status = func(args);
assert(status);
```
