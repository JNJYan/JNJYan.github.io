---
title: C++宏定义的那些事
date: 2021-03-23 21:57:57
tags: 
- C++ 
categories:
- C++相关
---


# 宏定义
宏定义将一个标识符定义为一个字符串，源代码中的标识符将被指定的字符串替换，这个过程发生在预处理阶段。
发方
C++程序的完整编译过程包括预处理、编译、汇编、链接四个步骤。其中预处理阶段进行宏展开、条件编译等。

在C语言中，通过宏来定义简单函数，可以降低函数调用过程中的开销，而在c++中，可以用`inline`来声明内联函数来降低函数调用过程的开销。

也可以通过宏来定义一些常量，但由于宏替换发生在预处理阶段，因此如果在编译过程中产生错误，很难进行错误的定位和溯源。

# 简单的宏定义
```c++
//第一种
#define PI2 3.1415926
#define Add(x) x+1 //错误示范
#define Double(x) (x*2) //错误示范
// Add(x) * 2 ==> x + 1 * 2
#define Add(x) ((x)+1)
#define Double(x) ((x)*2)


//第二种
#ifndef SOURCE_H
#define SOURCE_H

#define TEST
#ifdef TEST
std::cout << "this is a TEST " << endl;
#ifndef TEST
std::cout << "there are not TEST" << endl;
#endif
```
宏的两种简单用法如上所示，第一种用法将`PI`全部替换为`3.1415926`，这里的替换是直接展开的，如错误示范中所示，因此我们在使用宏定义时，要利用括号保证其在展开时不会产生错误。

第二种用法通过宏定义来实现条件编译、避免头文件的重复引入。

# 宏的高阶用法
`#`、`##`、`#@`三种可以称之为宏的高阶用法，可以实现许多高级特性。

```c++
#define STRING(msg) #msg
char* str = "hello";
char* str = STRING(hello);  //二者等价


#define CHAR(ch) #ch
char ch = 'a';
char ch = CHAR(a); //二者等价

#define 
```
当宏中遇到`#`和`##`时，是不能够进行嵌套替换的，不会对`#`和`##`之后宏进行展开。


