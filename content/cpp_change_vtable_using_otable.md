---
title: "Cpp change vtable"
date: 2023-07-11T21:35:28+08:00
draft: false
tags: []
slug: ""  # The URL last content: /usr/:1/:2/:slug{or :title}
mathjax: false
---

[Source code can be found here](https://github.com/jiaoshijie/code_misc/blob/main/c-like/vtables/change_vtable.cpp)

如果c++的类有虚函数那么这个类的实例对象就会有一个虚表. 我们可以通过一下代码来改变虚表.

```cpp
#include <cstring>
#include <iostream>

class A {
public:
  virtual void vfoo_1() { std::cout << "a_vfoo_1\n"; }

  virtual void vfoo_2(long long a) {
    std::cout << "a_vfoo_2: " << a << std::endl;
  }

  virtual int vfoo_3(int a, int b) { return a + b; }
};

typedef void (*void_pfunc_void)();
typedef void (*void_pfunc_int)(long long);
typedef int (*int_pfunc_int_int)(int, int);

struct other_vtable {
  void_pfunc_void pf1;
  void_pfunc_int pf2;
  int_pfunc_int_int pf3;
};

void g_f1() { std::cout << "g_f1\n"; }

void g_f2(long long a) {
  ((A *)a)->vfoo_1();
  printf("%p\n", (void *)a);
  // std::cout << "g_f2: " << a << std::endl;
}

int g_f3(int a, int b) { return b * 10; }

int main(void) {
  A *a = new A;
  other_vtable *ovt = new other_vtable;
  ovt->pf1 = g_f1;
  ovt->pf2 = g_f2;
  ovt->pf3 = g_f3;

  void *ptr = a;
  printf("%p\n", a);

  a->vfoo_1();
  a->vfoo_2(10);
  printf("a->vfoo_3: %d\n", a->vfoo_3(2, 5));
  puts("-------------------------------");

  memcpy(ptr, &ovt, sizeof(void *));

  a->vfoo_1();
  a->vfoo_2(10);
  printf("a->vfoo_3: %d\n", a->vfoo_3(2, 5));

  int breakpoint = 0;

  return 0;
}
```

内存布局大致如下图所示:

![cpp change vtable 01](../../images/cpp_change_vtable_01.png)

但是运行上面的代码可以发现在`memcpy(ptr, &ovt, sizeof(void *));`之后, `a->vfoo_1()`的输出是预期的输出, 但是`a->vfoo_2(10)`和`a->vfoo_3(2, 5)`部分的代码输出却很怪异.

这是因为编译这段代码时我们是以method的方式来执行这个函数的因此编译器除了将参数入栈外还会将this这个指针入栈(在参数之前入栈), 而我们执行的确实function因此函数会将this指针当作第一个参数, 将传入的第一个参数作为第二个参数(如果有的话), 所以可以看到`g_f2()`中的`((A *)a)->vfoo_1();`这行代码可以正常执行, `g_f3()`中返回值为20.

具体细节可以在[这个网站](https://godbolt.org/z/df3TTdjGj)查看汇编代码.

## Reference

- [Call stack](https://en.wikipedia.org/wiki/Call_stack)
