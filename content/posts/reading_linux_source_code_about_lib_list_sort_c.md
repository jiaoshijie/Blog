---
title: "分析 Linux 核心源代码 lib/list_sort.c"
date: 2024-03-23T14:33:19+08:00
tags: ["linux", "c"]
categories: ""
author: "Jiao shijie"
draft: true
hidemeta: true
math: true
---

```c
#define unlikely(x)    (__builtin_expect(!!(x), 0))
#define likely(x)    (__builtin_expect(!!(x), 1))
```

<++>
