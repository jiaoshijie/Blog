---
title: "'二维数组', '指针数组'和'指针的指针'的内存布局"
date: 2020-05-13T23:00:33+08:00
draft: false
tags: ["c"]
hidemeta: true
math: false
---

## 开始

下面提到的变量均是在 main 函数中申请的变量。

[文中代码](https://github.com/jiaoshijie/code_misc/tree/main/c-like/doubleArray_pointer2pointer)

### 1. 二维数组

首先讨论二维数组, 他在内存中的布局是最简单的, 考虑如下变量

```c
char str[][5] = {
  "jiao",
  "_shi",
  "_jie",
};
```

可以通过如下代码打印各个部分的地址

```c
printf("str addr: %p\n\n", &str);
for (int i = 0; i < 3; i++)
{
  printf("str[%d] addr: %p\t", i, &str[i]);
}
printf("\n\n");
for (int i = 0; i < 3; i++)
{
  for (int j = 0; j < 5; j++)
  {
    printf("str[%d][%d] addr: %p\n", i, j, &str[i][j]);
  }
  printf("\n");
}
```

可以看到地址从`str[0][0]`开始到结束是从低到高连续的。因为是在main函数中申请的
这个二维数组，因此它被存储在栈空间中(栈空间地址是高地址到低地址走的)，存储布局
大致如下：

![da_p2p_pa_01](../../images/da_p2p_pa_01.png#center)

### 2. 指针的指针

使用如下代码申请指针的指针的空间:

```c
char **d_ptr = (char **)malloc(sizeof(char *) * 3);
for (int i = 0; i < 3; i++)
{
  d_ptr[i] = (char *)malloc(sizeof(char) * 5);
}

memcpy(d_ptr[0], "jiao", 5);
memcpy(d_ptr[1], "_shi", 5);
memcpy(d_ptr[2], "_jie", 5);
```

使用如下代码输出各个部分的地址:

```c
printf("d_ptr addr: %p\n\n", &d_ptr);
for (int i = 0; i < 3; i++)
{
  printf("d_ptr[%d] addr: %p\n", i, &d_ptr[i]);
}
printf("\n\n");
for (int i = 0; i < 3; i++)
{
  for (int j = 0; j < 5; j++)
  {
    printf("d_ptr[%d][%d] addr: %p\n", i, j, &d_ptr[i][j]);
  }
  printf("\n");
}
```

可以看到只有d_ptr的地址是在栈空间中的, 其余部分均是在堆空间中.

因此指针的指针的内存布局如下:

![da_p2p_pa_02](../../images/da_p2p_pa_02.png#center)

### 3. 指针数组

对于指针数组情况比较复杂，首先需要清楚一个C程序在内存中的布局，大致布局可以参考下图

![da_p2p_pa_03](../../images/da_p2p_pa_03.png#center)

同时考虑以下简单的代码

```c
char *str1 = "hello";
char str2[] = "hello";
```

其中 `"hello"` 被称作 **string literal**，根据 C 标准的规定其会被放到一个静态的
存储空间中，而 gcc 会将其放到 `.rodata` 段中，这个段没有出现在上方的图中但也是
Static Memory Layout 中的一部分, 而且该区域的数据是不可以修改的。

> In translation phase 7, a byte or code of value zero is appended to each
> multibyte character sequence that results from a string literal or literals.
> The multibyte character sequence is then used to initialize
> an array of static storage duration and length just sufficient to contain
> the sequence. (C99 [6.4.5])

`str1` 被声明为一个指针，因此 `str1` 当中会存储一个地址而这个地址就是指向 `"hello"`。
因此 `str1` 实际指向了一个不允许被修改的地址(比如 `str1[0] = 'H'`)，当尝试修改
该位置的地址是为发生 Segment Falut Error。

而当对 `str2` 做同样的事情时(`str2[0] = 'H'`)，却有不同的结果，你会发现修改成功了。
这是因为 `str2` 被声明为了一个数组，它的数据应该被放置在栈空间。当程序每次执行时，
都会将 `.rodata` 中的内容，复制到栈空间中，因此你修改它没有任何问题。

而向下面一样申请的 `str` 是一个指针**数组**，它是一个数组然后每个元素都是一个指针，
如果要声明数组**指针**要 `char (*str)[N]` 这样 `str` 就是一个指向数组大小为 N 的
数据的指针。

基于上面的讨论我们可以知道`"jiao"`，`"_shi"` 和 `_jie`，都是在 `.rodata`
中的，而 `str` 又是一个指针**数组**，因此 `str[0]`，`str[1]` 和 `str[2]` 都是指向
了 `.rodata` 中的 **string literal**。

```c
char *str[] = {
  "jiao",
  "_shi",
  "_jie",
};
```

通过如下代码打印各个部分的地址:

```c
printf("str addr: %p\n\n", &str);
for (int i = 0; i < 3; i++)
{
  printf("str[%d] addr: %p\n", i, &str[i]);
}
printf("\n\n");
for (int i = 0; i < 3; i++)
{
  for (int j = 0; j < 5; j++)
  {
    printf("str[%d][%d] addr: %p\n", i, j, &str[i][j]);
  }
  printf("\n");
}
```

可以看到 `str[0]`，`str[1]`，`str[2]` 地址空间连续(偏移了一个指针的大小),
但是和 `str[0][0]` 地址相差很多，而 `str[0][0]`，`str[0][1]` 到 `str[2][4]`
地址空间都是连续的.

因此指针**数组**的内存布局如下:

![da_p2p_pa_04](../../images/da_p2p_pa_04.png#center)

读到这里，可以在返回去思考一下最开始提到的二维数组的内存布局，是不是会有不一样的想法。
