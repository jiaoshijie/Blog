---
title: "关于C二维数组, 二重指针, 指针数组的内存布局"
date: 2020-05-13T23:00:33+08:00
draft: false
tags: ["c"]
hidemeta: true
math: false
---

## 开始

前提: 在main函数中申请的各个变量, 不使用全局变量.

[一些测试代码在这里](https://github.com/jiaoshijie/misc_code/tree/main/badCode/cdoublepointer)

### 1. 二维数组

首先讨论二维数组, 他在内存中的布局是最简单的, 首先在main函数中申请如下变量.

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

可以看到地址从`str[0][0]`开始到结束是从低到高连续的. 因为是在main函数中申请的这个二维数组, 因此它被存储在栈空间中(栈空间地址是高地址到低地址走的), 存储布局大致如下:

![doublearray](../../images/cdoublearray.png)

### 2. 指针数组

对于指针数组首先做一个如下的简单申请:

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

可以看到str[0], str[1], str[2]地址空间连续(偏移了一个指针的大小), 但是和str[0][0]地址相差很多, 而str[0][0], str[0][1]到str[2][4]地址空间都是连续的.

这是因为指针数组是存储的指针类型, 而使用类似`"jiao"`方式来复制c会自动在堆空间中创建一块区域来存储该字符串, 因此str[0]中存储的是堆空间的地址(即str[0]指向堆空间).

因此指针数组的内存布局如下:

![cpointerarray01](../../images/cpointerarray01.png)

### 3. 双重指针

使用如下代码申请双重指针空间:

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

因此双重指针的内存布局如下:

![cdoublepointer01](../../images/cdoublepointer01.png)
