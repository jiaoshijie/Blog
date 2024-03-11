---
title: "关于数组array和&array地址相同问题"
date: 2020-05-17T16:28:28+08:00
draft: false
tags: ['c']
hidemeta: true
math: false
---

[文中代码](https://github.com/jiaoshijie/code_misc/tree/main/c-like/c_array_analysis)

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

可以看到输出的地址是不同的. 其实对于这一段代码是没有任何疑问的, 可以很轻松的
明白为什么输出的地址不同.

那么编译器是如何处理第一部分代码，使输出的地址相同的。

可以通过gdb调试可以很清晰的看到编译器是如何处理这种情况的。

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

可以看到不管是 `array` 还是 `&array` 的操作都是使用的 `lea` 操作将 `0x8(%esp)`
移动到 `%eax` 中然后在 `mov` 操作赋值到相关变量中.

而对于另一段代码:

```c
int **array = (int **)malloc(sizeof(int *) * 5);
for (int i = 0; i < 5; i++)
  array[i] = (int *)malloc(sizeof(int));
void *a1 = (void *)&array, *a2 = (void *)array;
```

```assembly
0x56556214 <main+103>:       lea    -0x2c(%ebp),%eax
0x56556217 <main+106>:       mov    %eax,-0x24(%ebp)
0x5655621a <main+109>:       mov    -0x2c(%ebp),%eax
0x5655621d <main+112>:       mov    %eax,-0x20(%ebp)
0x56556220 <main+115>:       mov    $0x0,%eax
```

可以看到在为 `a2` 赋值是使用的是 `mov` 指令而不是 `lea` 指令.

```assembly
;; mov 和 lea 指令的区别
;; 假设在地址`0x80`中有内容`1234`
mov 0x80, %eax // 执行完后eax内容为1234
lea 0x80, %eax // 执行完后eax内容为0x80
```

而编译器为什么会有这个行为呢？

> Except when it is the operand of the sizeof operator or the unary & operator,
> or is a string literal used to initialize an array,an expression that has
> type "array of type" is converted to an expression with type "pointer to type"
> that points to the initial element of the array object and is not an lvalue.
> If the array object has register storage class, the behavior is undefined. (C99 [6.3.2.1])
>
> A character string literal is a sequence of zero or more multibyte characters
> enclosed in double-quotes, as in "xyz". A wide string literal is the same,
> except prefixed by the letter L. (C99 [6.4.5])
>
> The multibyte character sequence is then used to initialize an array of static
> storage duration and length just sufficient to contain the sequence. (C99 [6.4.5])

第一段的主要意思是当一个`表达式`是一个数组并且操作它的操作符不是 `sizeof` 或
一元操作符 `&` 时，这个`表达式`会被转化为指向数组第一个元素的指针。下面两段是解释
为什么要提 **string literal** 的，有兴趣可以自己看看。

```c
int array[] = { 0, 1, 2, 3, 4 };
array == &array // is true
```

因此根据 C99 标准的说法，`array` 和 `&array` 虽然数值是相同的但表达的含义却不相同，
`array` 是指向数组第一个元素的指针，而 `&array` 为 `array` 这个 object 的地址。
