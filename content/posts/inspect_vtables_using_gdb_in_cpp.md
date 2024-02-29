---
title: "Inspect cpp vtables using gdb"
date: 2023-07-09T21:18:46+08:00
draft: false
tags: ['cpp', 'vtable']
hidemeta: true
math: false
---

[Source code can be found here(文章中的源代码)](https://github.com/jiaoshijie/code_misc/tree/main/c-like/vtables)

## 1. 使用`info vtbl obj`查看vtable

### vtable function

```cpp
class A {
public:
  A() = default;
  ~A() = default;

  virtual void vfoo_1() {}
  virtual void vfoo_2() {}
};

int main(void) {
  A *a = new A;
  int breakpoint = 0;
  return 0;
}
```

当一个class有virtual function时, 这个类会有一个vtable(virtual table)来记录这些函数的入口.

编译上面的代码, 使用gdb来debug这个程序, 使用`info vtbl a`可以可视化该实例对象a的vtable. 如下图所示:

![vtable_01](../../images/cpp_vtables_01.png)

### vtable class

```cpp
//    A
//   / \
//  /   \
// B     C
// \    /
//  \  /
//   D

class A {
public:
  ~A() = default;
  virtual void a_foo() {}
};

class B : virtual public A {
public:
  virtual void b_foo() {}
};

class C : virtual public A {
public:
  virtual void c_foo() {}
};

class D : public B, public C {
public:
};

int main(void) {
  D *d = new D;
  int breakpoint = 0;
  return 0;
}
```

同样使用`info vtbl d`可以查看实例对象d的vtable. 如下图所示:

![vtable_02](../../images/cpp_vtables_02.png)

## 2. 更近近一步观察

通过`info vtbl obj`可以确实可以查看vtable的信息, 但它在内存中究竟是如何存储的, 让我们来深入地讨论一下.

在讨论之前先看一段c代码:

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
  int *p = (int *)malloc(sizeof(int) * 10);
  for (int i = 0; i < 10; i++) {
    p[i] = i + 1;
  }

  for (int i = 0; i < 10; i++) {
    printf("%p: %d\n", &p[i], p[i]);
  }

  puts("----------------------------------------");

  for (int i = 0; i < 10; i++) {
    printf("%p: %d\n", &((void **)p)[i], ((void **)p)[i]);
  }

  puts("----------------------------------------");

  for (int i = 0; i < 10; i++) {
    printf("%p: %p\n", &((void **)p)[i], ((void **)p)[i]);
  }

  int breakpoint = 0;

  return 0;
}
```

这段代码声明了一个p指针(8 bytes), 然后使用malloc初始化了10个int(4 bytes)大小的空间.

所以, 此时p是一个指针指向10个int大小的空间.

`((void **)p)`这个代码将p由一个指针转化为一个二重指针, 也就是说将p指向的10个int大小的空间也按地址来处理, 也就是说原本10个大小的int空间会变成5个大小的地址空间.

所以这个程序的输出为:

![vtables_03](../../images/cpp_vtables_03.png)

### vtable function

让我们继续使用上面vtable function部分的代码来一步步的看. 使用gdb调试该程序.

- 使用`p *a`来输出该实例中的内容, 可以得到以下输出
  ```
  $3 = {_vptr.A = 0x555555557d90 <vtable for A+16>}
  ```
- 这个地址`0x555555557d90`就是a的vtable入口
- 使用`x /32xb 0x555555557d90`来查看该地址处的内容, 可得到如下输出
  ```
  0x555555557d90 <_ZTV1A+16>:     0x6e    0x51    0x55    0x55    0x55    0x55    0x00    0x00
  0x555555557d98 <_ZTV1A+24>:     0x7a    0x51    0x55    0x55    0x55    0x55    0x00    0x00
  0x555555557da0 <_ZTI1A>:        0x90    0xac    0xe6    0xf7    0xff    0x7f    0x00    0x00
  0x555555557da8 <_ZTI1A+8>:      0x04    0x60    0x55    0x55    0x55    0x55    0x00    0x00
  ```
- 可以到将第一行的8个字节组合起来可以得到和使用`info vtbl a`同样的地址`0x55555555516e`
- 使用`info symbol 0x55555555516e`, 可以得到以下输出
  ```
  A::vfoo_1() in section .text of /home/jsj/Downloads/Code/my/code_misc/c-like/vtables/runable
  ```
- 可以看到这个地址指向的就是a的第一个虚函数.
- 同理可以查看a的第二个虚函数.

### vtable class

但是使用上面的方法有时却不能看上一部分的vtable class代码的虚函数. 根据[这篇Wiki](https://en.wikipedia.org/wiki/Virtual_inheritance)的说法d应该有两个vtables. 但使用`p *a`却只能得到一个vtable入口, 如下所示
```
$3 = {<B> = {<A> = {_vptr.A = 0x555555557c08 <vtable for D+32>}, <No data fields>}, <C> = {<No data fields>}, <No data fields>}
```
而通过上面的步骤查看地址`0x555555557c08`处的虚函数会发现没有C的虚函数. 而可以通过下面的步骤来查看C的虚函数

- `p /a ((void **)d)[0]@10`, 输出如下
  ```
  $4 = {0x555555557c08 <_ZTV1D+32>, 0x555555557c38 <_ZTV1D+80>, 0x0, 0xed41, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
  ```
- 可以看到第一个地址就是上面的地址, 这里我们看第二个地址`0x555555557c38`
- 使用`x /32xb 0x555555557c38`查看该地址的内容如下
  ```
  0x555555557c38 <_ZTV1D+80>:     0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
  0x555555557c40 <_ZTV1D+88>:     0x86    0x51    0x55    0x55    0x55    0x55    0x00    0x00
  0x555555557c48 <_ZTT1D>:        0x08    0x7c    0x55    0x55    0x55    0x55    0x00    0x00
  0x555555557c50 <_ZTT1D+8>:      0xa0    0x7c    0x55    0x55    0x55    0x55    0x00    0x00
  0x555555557c58 <_ZTT1D+16>:     0xa0    0x7c    0x55    0x55    0x55    0x55    0x00    0x00
  ```
- 我们可以看到第二行的地址就是C的虚函数`info symbol 0x555555555186`
  ```
  C::c_foo() in section .text of /home/jsj/Downloads/Code/my/code_misc/c-like/vtables/runable
  ```

## 3. 另一种直接查看vtable的方法

继续使用vtable class代码使用`p /a (*(void ***)d)[0]@10`可以看到如下输出:

![vtables_04](../../images/cpp_vtables_04.png)

同样使用`p /a (*(void ***)a)[0]@10`可以查看vtable function代码的虚函数:

![vtables_05](../../images/cpp_vtables_05.png)


## 4. 解释 `p /a ((void **)a)[0]@10` 和 `p /a (*(void ***)a)[0]@10` 含义


![vtables_06](../../images/cpp_vtables_06.png)

1. 如果一个类有vtable, 则结构大致如上图所示, 结合第2部分所说`((void **)a)`就是将a转换为二重指针所以在处理图中中间那一块的数据时会将其当作地址来处理, 所以`((void **)a)[0]`就相当与`*((void **)a)`所以会输出`0x333`而`((void **)a)[1]`相当于`*((void **)a) + 1`会打印0x333下面那一块的内容(图中并没有画出). 所以使用这个方法可以打印出vtable的入口(即便是有多个入口也可以打印出来, 比如上面的vtable class).
2. `p /a (*(void ***)a)[0]@10`: `(void ***)a`将a转化为一个三重指针, 也就是说最右边的部分的数据也会被当作地址来处理. `*(void ***)a` 对a进行一次解引用(dereference), 也就是说现在`*(void ***)a`是图中的vtable ptr, 然后`(*(void ***)a)[0]` 也就相当于`*(*(void ***)a)` 因此会输出0x444, 而`(*(void ***)a)[1]`就相当与`*(*(void ***)a) + 1`所以会输出0x444下面的内容(图中并没有画出). 因此这个命令可以直接打印出虚函数的地址(入口).

## References

- [Virtual inheritance](https://en.wikipedia.org/wiki/Virtual_inheritance)
- [digging-into-c-vtables](https://levelup.gitconnected.com/digging-into-c-vtables-572f23aa9d43)
- [why-to-cast-pointer-to-double-pointer-in-c](https://stackoverflow.com/questions/63514129/why-to-cast-pointer-to-double-pointer-in-c)
