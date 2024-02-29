---
title: "关于数组array和&array地址相同问题"
date: 2020-05-17T16:28:28+08:00
draft: false
tags: ['c']
hidemeta: true
math: false
---

## 开始

[测试代码](https://github.com/jiaoshijie/misc_code/tree/main/badCode/c_array_analysis)

对于如下代码:

```c
int array[] = { 0, 1, 2, 3, 4 };
printf("%p\n", &array);
printf("%p\n", array);
```

可以看到输出的地址是相同的.

而对于如下代码:

```c
int **array = (int **)malloc(sizeof(int *) * 5);
for (int i = 0; i < 5; i++)
  array[i] = (int *)malloc(sizeof(int));
printf("%p\n", &array);
printf("%p\n", array);
```

可以看到输出的地址是不同的. 其实对于这一段代码是没有任何疑问的, 可以很轻松的明白为什么输出的地址不同.

令我好奇的是编译器是如何处理第一部分代码, 使输出的地址相同的.

其实, 我也知道array和&array的地址在栈空间中是相同的. 但我想知道编译器是如何处理这种情况的.

通过gdb调试可以很清晰的看到编译器, 是如何处理这种情况的.

```c
int array[] = { 0, 1, 2, 3, 4 };
void *a1 = (void *)&array, *a2 = (void *)array;
```

可以看到void这一行的汇编代码为:

```as
0x565561e4 <main+71>:        lea    0x8(%esp),%eax
0x565561e8 <main+75>:        mov    %eax,(%esp)
0x565561eb <main+78>:        lea    0x8(%esp),%eax
0x565561ef <main+82>:        mov    %eax,0x4(%esp)
0x565561f3 <main+86>:        mov    $0x0,%eax
```

可以看到不管是array还是&array的操作都是使用的`lea`操作将`0x8(%esp)`移动到`%eax`中然后在`mov`操作赋值到相关变量中.

而对于另一段代码:

```c
int **array = (int **)malloc(sizeof(int *) * 5);
for (int i = 0; i < 5; i++)
  array[i] = (int *)malloc(sizeof(int));
void *a1 = (void *)&array, *a2 = (void *)array;
```

```as
0x56556214 <main+103>:       lea    -0x2c(%ebp),%eax
0x56556217 <main+106>:       mov    %eax,-0x24(%ebp)
0x5655621a <main+109>:       mov    -0x2c(%ebp),%eax
0x5655621d <main+112>:       mov    %eax,-0x20(%ebp)
0x56556220 <main+115>:       mov    $0x0,%eax
```

可以看到在为a2赋值是使用的是`mov`指令而不是`lea`指令.

## mov 和 lea 指令的区别

假设在地址`0x80`中有内容`1234`

```as
mov 0x80, %eax // 执行完后eax内容为1234
lea 0x80, %eax // 执行完后eax内容为0x80
```
