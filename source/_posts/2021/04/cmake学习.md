---
title: 现代cmake学习
date: 2021-04-09 15:31:38
tags:
- cmake
categories:
- 编译
---

# Introduction
CMake是一个编译系统生成工具，而非编译系统。CMake能够生成编译系统的输入文件如`Makefile`，CMake本身支持`Make/Ninja/Visual Studio/XCode`等。

CMake是跨平台的，支持Linux、Windows、OSX等，同时也支持跨平台构建（编译器要支持跨平台才可以哦）。

CMake开始于1999/2000年，现代CMake开始于2014年的`3.0`版本，现代CMake有一个非常重要的概念，`Everything is a (self-contained) target`。

> Everything that is needed to (successfully) build that target.
>  
> Everything that is needed to (successfully) use that target.

# Let's Go
时刻牢记以下三句话。

- Declare a target
- Declare target's traits
- It's all about targets

## Minimum Version

```cmake
cmake_minimum_required(VERSION 3.5)
```


## Setting a Project
```cmake
project(hello-world VERSION 1.0
                    DESCRIPTION ""
                    LANGUAGES CXX) # C/CXX/ASM/CUDA/FORTAN/SWIFT

add_subdirectory(src)

install(FILES COPYRIGHT README.md DESTINATION share/doc/cmake/hello-world)
install(PROGREAMS **.sh DESTINATION bin)
install(DIRECTORY doc DESTINATION share/doc/cmake/hello-world)
```

## Making an Executable
```cmake
add_executable(one two.cpp three.h)
```
`one`既是生成的可执行程序也是Target，后续为源文件列表和头文件列表，大多数情况下，头文件将会被忽略，只是为了让他们显示在IDE中。

## Making an Library
```cmake
# BUILD_SHARED_LIBS
add_library(one two.cpp three.h)
# 静态库
add_library(one STATIC two.cpp three.h)
# 动态库
add_library(one SHARED two.cpp three.h)
# 模块
add_library(one MODULE two.cpp three.h)
```

## Target
```cmake
target_include_directories(ont PUBLIC include)
target_include_directories(ont PRIVATE include)
target_include_directories(ont INTERFACE include)
```

`PUBLIC`意味着所有链接到此目标的目标都需要包含`include`目录，`PRIVATE`表示只有当前target需要，依赖项不需要，`INTERFACE`表示只有依赖项许需要。

```c++
add_library(another STATIC another.cpp another.h)
target_link_libraries(another PUBLIC one)
```
- `target_include_directories`指定了target包含的头文件路径。

- `target_link_libraries`指定了target链接的库。

- `target_compile_options`指定了taget的编译选项。

target由`add_library()`和`add_executable()`生成。

我们以如下工程目录介绍`PUBLIC/PRIVATE/INTERFACE`。

```shell
cmake-test/                 工程主目录，main.c 调用 libhello-world.so
├── CMakeLists.txt
├── hello-world             生成 libhello-world.so，调用 libhello.so 和 libworld.so
│   ├── CMakeLists.txt
│   ├── hello               生成 libhello.so 
│   │   ├── CMakeLists.txt
│   │   ├── hello.c
│   │   └── hello.h         libhello.so 对外的头文件
│   ├── hello_world.c
│   ├── hello_world.h       libhello-world.so 对外的头文件
│   └── world               生成 libworld.so
│       ├── CMakeLists.txt
│       ├── world.c
│       └── world.h         libworld.so 对外的头文件
└── main.c
```

其调用关系如下所示

```shell
                                 ├────libhello.so
可执行文件────libhello-world.so
                                 ├────libworld.so
```

`PRIVATE`:生成`libhello-world.so`时，只在`hello_world.c`中包含了`hello.h`，`libhello-world.so`对外的头文件`hello_world.h`不包含`hello.h`，并且`main.c`不调用`hello.c`中的函数，那么应当用`PRIVATE`。

```cmake
target_link_libraries(hello-world PRIVATE hello)
target_include_directories(hello-world PRIVATE hello)
```

`INTERFACE`:生成`libhello-world.so`时，只在`libhello-world.so`对外的头文件`hello_world.h`包含`hello.h`，`hello_world.c`不包含`hello.h`即`libhello-world.so`不使用`libhello.so`提供的功能，只需要`hello.h`中定义的结构体/类等类型信息，但`main.c`需要调用`hello.c`中的函数即`libhello.so`中的函数，那么应当用`INTERFACE`。

```cmake
target_link_libraries(hello-world INTERFACE hello)
target_include_directories(hello-world INTERFACE hello)
```

`PUBLIC`:生成`libhello-world.so`时，在`libhello-world.so`对外的头文件`hello_world.h`包含`hello.h`，`hello_world.c`也包含`hello.h`即`libhello-world.so`使用`libhello.so`提供的功能，并且`main.c`需要调用`hello.c`中的函数即`libhello.so`中的函数，那么应当用`PUBLIC`。

```cmake
target_link_libraries(hello-world PUBLIC hello)
target_include_directories(hello-world PUBLIC hello)
```

着重理解**依赖传递**的概念，`main.c`依赖于`libhello-world.so`，`libhello-world.so`依赖于`libhello.so`和`libworld.so`，若`main.c`不调用`libhello.so`中的功能，则`hello-world`与`hello`之间采用`PRIVATE`。若`main.c`调用`libhello.so`中的函数，但`libhello-world.so`不调用，则用`INTERFACE`。若`main.c`和`libhello-world.so`都调用`libhello.so`的函数，则使用`PUBLIC`关键字。

可以参考C++继承中`PRIVATE/PROTECTED/PUBLIC`的概念[<sup>1</sup>](#leimao)。
## Variables
### Local Variables
```cmake
set(VAR1 "local variable")

message("VAR1 is : " ${MY_VARIABLE})
```

### Cache Variables
```cmake
set(VAR2
    belebele1 CACHE STRING "cache")
message("VAR2 is : " ${VAR2})
```

命令行调用之后，会将该变量写入`CMakeCache.txt`，之后调用若不从命令行重新赋值，则会一直采用Cache中的值。

```shell
mkdir build && cd build
cmake .. -DMY_CACHE_VARIABLE(:STRING)=belebele
cmake -L .. # 列出当前cache变量
```

`bool`类型的变量常用OPTION表示，OPTION也可以看作cache变量的一种，所以会写进`CMakeCache.txt`。

```cmake
OPTION(VAR3 "description" OFF/ON)
```

cmake的一些常见变量见官网[<sup>2</sup>](#cmake-variable)。

### Environment Variables
```cmake
set(ENV{variable_name} value)
$ENV{variable_name}
```

### Properties
```cmake
set_property(TARGET TargetName PROPERTY CXX_STANDARD 11)

set_target_properties(TargetName PROPERTY CXX_STANDARD 11)

get_property(Result TARGET TargetName PROPERTY CXX_STANDARD)
```

## Control Flow
```cmake
if(${variable})
    xxx
else()
    zzz
endif()
```

## Function
```cmake
function(SIMPLE_FUNC)
    message("simple function")
endfunction()
```

其他控制逻辑有`NOT/TARGET/EXISTS/DEFINED/AND/OR/STREQUAL/MATCHES/VERSION_LESS/VERSION_LESS_EQUAL`等。

# 参考文献

<div id="modern-cmake"></div>

- [moder-cmake](https://cliutils.gitlab.io/modern-cmake/)

<div id="youtube"></div>

- [youtube/c++2018](https://www.youtube.com/watch?v=y7ndUhdQuU8)

<div id="leimao"></div>

- [CMake-Public-Private-Interface](https://leimao.github.io/blog/CMake-Public-Private-Interface/)

<div id="intheritance"></div>

- [Modern CMake is like inheritance](https://kubasejdak.com/modern-cmake-is-like-inheritance)

<div id="cmake-variable"></div>

- [cmake-variable](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html)