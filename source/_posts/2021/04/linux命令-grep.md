---
title: linux命令-grep
date: 2021-04-08 10:36:49
tags:
- linux
categories:
- 每天一个linux命令
---

# grep简介
grep命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。grep全称是Global Regular Expression Print，表示全局正则表达式版本，它的使用权限是所有用户。

grep在一个或多个文件中搜索字符串模板，如果模板包括空格，则必须使用引号引用，模板后的所有字符串被看作文件名。搜索的结果被送到标准输出，不影响原文件内容。

grep可用于shell脚本，因为grep通过返回一个状态值来说明搜索的状态，如果模板搜索成功，则返回0，如果搜索不成功，则返回1，如果搜索的文件不存在，则返回2。我们利用这些返回值就可进行一些自动化的文本处理工作。

shell中可以通过`$?`获取上一个命令的返回值。

# grep用法

```text
Usage: grep [OPTION]... PATTERN [FILE]...
Search for PATTERN in each FILE.
Example: grep -i 'hello world' menu.h main.c
```

# grep参数
|-参数|--参数|用途|
|---|---|---|
|-a|--text|搜索二进制数据|
|-i||