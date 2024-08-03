---
title: "Ctags_usage"
date: 2024-08-04T00:18:07+08:00
tags: [""]
categories: ""
author: "Jiao shijie"
draft: false
hidemeta: true
math: false
---

`ctags gun/global`

## C and C++

### 1. 通常使用 ctags

- `ctags -R . -f .tags`
  - `-R` 从当前目录递归开始生成 ctags 索引
  - `.` 当前目录
  - `-f` `-o` 指定生成的文件名
- `ctags -R --c-kinds=+p --fields=+iaS --extra=+q . -f .tags` 获取更多的信息

### 2. 一些 ctags 基础参数

1. ctags 支持多种语言的索引
   - `ctags --list-maps` 查看支持的语言种类
2. ctags 支持很多语义但并不是默认都开启的
   - `ctags --list-kinds` 查看所有语言支持的语义及默认开启状态(后面有 `off` 表示默认没有开启)
   - `ctags --list-kinds={lang}` 查看{lang}语言支持的语义及开启状态
   - 使用 `ctags --{lang}-kinds=+px` 来开启默认不支持的语义
3. `ctags --langmap={lang}:+.inl` 可以指定某种扩展名也为该 lang 的源文件
4. `ctags -h +.inc` 指定 `.inc` 文件也为头文件
5. `ctags --fields=+aiKSz` 如何描述语法元素
   - `a` 表示如果是类的成员, 要标明其 access 属性(即是 public 的还是 private 的)
   - `i` 表示如果有继承, 要标明父类
   - `S` 表示如果是函数, 要标明函数的 signature
   - `K` 表示要显示语法元素类型的全称
   - `z` 表示在显示语法元素的类型时, 使用格式 kind:type
6. `ctags --extras=+q` 要求 ctags 记录全名(类和函数名), 可以更精确的定位
7. `ctags --exclude=lex.yy.cc` 不要扫描某些目录或文件

### 3. ctags 使用

1. 使用`ctags -R --c-kinds=+p --fields=+iaS --extras=+q /usr/include` 将可以创建外部的跳转, 但会使用很长的时间, 得到的文件一般也很大

2. 可以只创建当前文件包含的头文件的索引

``` sh
#! /bin/bash

# ./ctags_with_dep.sh file1.c file2.c -Ifreetype ... to generate a tags file for these files.

gcc -M "$@" | sed -e 's/[\\ ]/\n/g' | \
        sed -e '/^$/d' -e '/\.o:[ \t]*$/d' | \
        ctags -R -L - --c++-kinds=+p --fields=+iaS --extras=+q -f .rtags
```

### Example

```sh
# 这将创建 dwm 的外部文件索引
gcc -M -I/usr/include/freetype2 dwm.c | \
        sed -e 's/[\\ ]/\n/g' | \
        sed -e '/^$/d' -e '/\.o:[ \t]*$/d' | \
        ctags -L - --c-kinds=+p -f .rtags
```

## python

- `ctags -R --fields=+l --languages=python --python-kinds=-iv -f ./tags ./`
- [python use ctags with vim](https://www.codeunderscored.com/list-the-python-kinds-in-ctags/)

## 参考

- [Generate Ctags files for c/c++](https://www.topbug.net/blog/2012/03/17/generate-ctags-files-for-c-slash-c-plus-plus-source-files-and-all-of-their-included-header-files/)
