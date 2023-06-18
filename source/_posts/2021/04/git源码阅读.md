---
title: git源码阅读
date: 2021-04-07 16:06:24
tags: 
- git
categories:
- 源码阅读
---

## git源码获取

<https://github.com/git/git>

```shell
## 查看第一个提交
git log --date-order --reverse
## 切换至第一个提交
git checkout e83c5163316f89bfbde7d9ab23ca2e25604af290
```

## 源码编译

修改Makefile

```shell
-LIBS= -lssl
+LIBS= -lssl -lz -lcrypto
```

编译

```shell
make
```

## 可执行程序

- init-db
- update-cache
- read-tree
- write-tree
- cat-file
- commit-tree
- show-diff

### init-db

`init-db`同现代git中的`git init .`，在当前目录初始化仓库。

1. 创建目录`.dircache`。
2. 创建目录`.dircache/objects`。
3. 在`.dircache/objects`下创建目录`00~ff`共256个目录。

### update-cache

将工作区的修改提交到暂存区。

1. 读取并解析索引文件`.dircache/index`。
2. 遍历多个文件，读取并生成变更文件信息(文件名称、文件内容sha1值、日期、大小等)，写入到索引文件中。
3. 遍历多个文件，读取并压缩变更文件，存储到objects文件中，该文件为blob对象。

### cat-file

根据sha1值查看暂存区中的objects文件内容。objects内容为压缩格式，基于zlib压缩算法。

1. 根据传入的sha1值定位objects文件。
2. 读取该objects文件内容，解压得到真实数据。
3. 写入到临时文件`temp_git_file_xxxx`中。
