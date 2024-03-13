---
title: "快速计算正整数的平方根"
date: 2024-03-11T20:09:17+08:00
tags: ["c", "math"]
categories: ""
author: "Jiao shijie"
draft: true
hidemeta: true
math: true
---

如何求一个正整数的平方根，最直接的方法就是

```c
int i_sqrt(int N)
{
  int res = 1;
  while (res * res <= N) {
    res++;
  }
  return res - 1;
}
```

但上面的计算方法非常的低效，而对于求正整数的平方根，已经有非常多的算法。

下面介绍一个常用的正整数开平方根的算法。

## Digit by digit calculation

[Digit by digit calculation](https://en.wikipedia.org/wiki/Methods_of_computing_square_roots#Digit-by-digit_calculation)

假设 $x = \sqrt{N}$ 则 $x^2 = N$ 以二进制表示 $x$，则 $x^2$ 为

$$
x^2 = (000b_0b_1b_2 \cdots b_{n-1}b_n)^2 \tag{1}
$$

其中，$b_0$ 为 $x$ 的二进制表示中第一个为 1 的二进制位。但要注意 $b_1 \cdots b_n$
并比一定全为 1。我们可以将公式 (1) 该为加法形式。

$$
x^2 = (a_n + a_{n-1} + \cdots + a_1 + a_0)^2 \tag{2}
$$

其中，$a_m = 2^m$ 或 $a_m = 0$，取决于对应二进制位的值为 1 还是 0。将其展开可以得到

$$
\begin{bmatrix}
 a_0a_0     & \cdots & a_0a_{n-2}     & a_0a_{n-1}     & a_0a_{n}   \\\
 \vdots     & \ddots & \vdots         & \vdots         & \vdots     \\\
 a_{n-2}a_0 & \cdots & a_{n-2}a_{n-2} & a_{n-2}a_{n-1} & a_{n-2}a_n \\\
 a_{n-1}a_0 & \cdots & a_{n-1}a_{n-2} & a_{n-1}a_{n-1} & a_{n-1}a_n \\\
 a_na_0     & \cdots & a_na_{n-2}     & a_na_{n-1}     & a_na_n
\end{bmatrix}
$$

仔细观察可以发现

$$
\begin{bmatrix}
a_{n-2}a_{n-2} & a_{n-2}a_{n - 1} & a_{n-2}a_n \\\
a_{n-1}a_{n-2} &                  &            \\\
a_na_{n-2}     &                  &
\end{bmatrix}
$$

$$
\begin{bmatrix}
a_{n-1}a_{n-1} & a_{n-1}a_n \\\
a_na_{n-1}     &
\end{bmatrix}
$$

$$
\begin{bmatrix}
a_na_n
\end{bmatrix}
$$

因此，$x^2$ 可以整理成如下形式

---

$$
x^2 = a_n^2 + a_{n-1}(2a_n + a_{n-1}) + a_{n-2}(2(a_n + a_{n-1}) + a_{n-2}) + \cdots +\\\
a_0(2\sum_{i=1}^{n}a_i + a_0) \tag{3}
$$

---

令 $P_m = a_n + a_{n-1} + \cdots + a_m$，其中 $x = a_n + a_{n-1} + \cdots + a_0$。

显然 $P_m = P_{m + 1} + a_m$，我们的目标是试出所有的 $a_m$ ($a_m = 2^m$ 或 $a_m = 0$)，
就是从 $m = n$ 试到 $m = 0$，如果 $P_m^2 \le N$，则 $a_m = 2^m$，否则 $a_m = 0$。公式如下

$$
\begin{cases}
 a_m = 2^m & \text{, if } P_m^2 \le N \\\
 a_m = 0 & \text{, Otherwise }
\end{cases}
$$

但是如果每次都比较 $P_m^2 \le N$，程序的效率依旧很低，因此我们可以根据公式(3)
定义 $X_m$ 为 $N$ 与 $P_m^2$ 差值，使用 $X_m = X_{m+1} - Y_m$ 来更新 $X_m$，$X_m$
和 $Y_m$的定义如下所示

$$
X_m = N - P_m^2 = X_{m+1} - Y_m
$$

$$
Y_m = a_m(2P_{m+1} + a_m) = 2P_{m+1}a_m + a_m^2 = 2^{m+1}P_{m+1} + (2^{m})^2
$$

再对 $Y_m$ 进行划分

$$
c_m = 2P_{m+1}a_m = 2^{m+1}P_{m+1}
$$

$$
d_m = (2^{m})^2 = 4^m
$$

$$
Y_m = \begin{cases}
c_m + d_m & \text{, if } a_m = 2^m \\\
0         & \text{, if } a_m = 0
\end{cases}
$$

而 $c_m$ 和 $d_m$ 可以被很简单的更新

$$
c_{m-1} = 2^{m}P_{m} = \frac{2^{m+1}}{2}(P_{m+1} + a_{m}) = \begin{cases}
c_m / 2 + d_m & \text{, if } a_m = 2^m \\\
c_m / 2       & \text{, if } a_m = 0
\end{cases}
$$

$$
d_{m-1} = \frac{d_m}{4}
$$

**注意** 根据定义 $c_{-1} = 2^0P_0 = P_0 = x = \sqrt{N}$。

因此可以写出如下代码

```c
#include <math.h>
#include <assert.h>

int i_sqrt(int N) {
    assert("N must be greater than 0" && N > 0);
    // 计算出二进制位最高位为1的bit位置
    int n = log2(N);

    int X = N;  // X_m = N - P_m^2, m = n + 1, P_m^2 = 0 => X_{n+1} = N

    int c = 0;  // c_m = 2P_{m+1}a_m, m = n => c_n = 0
    long d = 1 << (n << 1);  // d_m = a_m^2, m = n => d = (2^n)^2 = 2^{2n}

    while (d) {  // d_n ... d_0
        if (X >= c + d) {  // X{m+1} - Y_m >= 0
            X -= c + d;    // a_m = 2^m then X_{m+1} - Y_m
            c = (c >> 1) + d;
        } else {  // Y_m = 0
            c >>= 1;
        }
        d >>= 2;
    }

    return c;
}
```

但上面这段代码依然有优化的空间，首先其依赖 `double log2(double)` 来计算最高位
bit为1的位置，导致编译时需要指定 `-lm`，且 `log2` 本身是做到浮点运算，相对较消耗
计算资源。

而 gcc 有一个 builtin 的函数 `int __builtin_clz(unsigned int x)` 可以计算从高位到
低位连续0的个数。

[__builtin_clz](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html)

> Built-in Function: int __builtin_clz (unsigned int x)
>
> Returns the number of leading 0-bits in x, starting at the most significant
> bit position. If x is 0, the result is undefined.

因此我们可以通过 `31 - __builtin_clz(x)` 来得到最高位第一个为1的位置，来替换
`log2` 函数。同时上面代码 $d_m$ 的计算也可以优化。

考虑到 $d_m = (2^m)^2$ 及下面判断 $X_{m+1} >= c_m + d_m$，当 $m = n$ 时，$X_{n+1} = N$
，$c_n = 0$，$d_n = (2^n)^2$，除了 $N = 1$ 的情况，$d_n$ 一定大于 $N$，也就是我们
在更新的时候就只是在更新 $d_m$ 直到 $d_m <= N$ 才会更新 $c_m$。因此我们也不必严格
按照上面公式定义初始化 $d_m$，可以进行一些优化。考虑 $N$ 的最高位为1的索引为 $t$，
无论比 $t$ 低的位数有多少1，$2^{t+1}$ 一定大于 $N$，因此初始化 $d_m$ 时找到 $2m \le t$，
同时 $2m$ 是一定为偶数的。

因此可以将上面代码中初始化 `d` 的代码改为
```c
// int n = log2(N);  // this line is not needed anymore.
int d = 1UL << ((31 - __builtin_clz(N)) & ~1UL);
```

因为 $2m \le t$ 因此可以将 `d` 的类型变为 `int`，`~1UL` 是为了保证结果为偶数。

现在这段代码不在依赖 `math.h` 了，但依然依赖 gcc，因为有使用 gcc 提供的函数
`__builtin_clz`，因此这段代码在 Microsoft 的编译器上可能编译不通过。

因此我们可以尝试自己实现一个类似 `int ffs(int i)` (`man ffs`)的函数 `fls`
(find last bit set in a word)，最直接的实现方法就是下面的代码

```c
int fls(int word) {
  int res = 31;  // 0-based index
  int mask = 1 << 31;

  while (!(word & mask)) {
    res--;
    mask >>= 1;
  }

  return res;
}
```

还可以对以上代码进一步优化

```c
int fls(int word) {
    int res = 31;  // 0-based index
    if (!(word & 0xffff0000)) {
        res -= 16;
        word <<= 16;
    }

    if (!(word & 0xff000000)) {
        res -= 8;
        word <<= 8;
    }

    if (!(word & 0xf0000000)) {
        res -= 4;
        word <<= 4;
    }

    if (!(word & 0xc0000000)) {
        res -= 2;
        word <<= 2;
    }

    if (!(word & 0x80000000))
        res -= 1;

    return res;
}
```

完整代码如下，已经不需要任何依赖了。🎉🎉🎉

```c
#include <assert.h>

int fls(int word) {
    int res = 31;  // 0-based index
    if (!(word & 0xffff0000)) {
        res -= 16;
        word <<= 16;
    }

    if (!(word & 0xff000000)) {
        res -= 8;
        word <<= 8;
    }

    if (!(word & 0xf0000000)) {
        res -= 4;
        word <<= 4;
    }

    if (!(word & 0xc0000000)) {
        res -= 2;
        word <<= 2;
    }

    if (!(word & 0x80000000))
        res -= 1;

    return res;
}

int i_sqrt(int N) {
    assert("N must be greater than 0" && N > 0);
    int X = N;  // X_m = N - P_m^2, m = n + 1, P_m^2 = 0 => X_{n+1} = N

    int c = 0;  // c_m = 2P_{m+1}a_m, m = n => c_n = 0
    int d = 1UL << (fls(N) & ~1UL);  // d_m = a_m^2, m = n => d = (2^n)^2 = 2^{2n}

    while (d) {  // d_n ... d_0
        if (X >= c + d) {  // X{m+1} - Y_m >= 0
            X -= c + d;    // a_m = 2^m then X_{m+1} - Y_m
            c = (c >> 1) + d;
        } else {  // Y_m = 0
            c >>= 1;
        }
        d >>= 2;
    }

    return c;
}
```

## 快速计算正整数平方根的应用

读到这，你可能会想为什么需要快速计算正整数平方根，它的应用在哪里？

- TODO

## 其它相关算法

- [lookup table square root](https://web.ece.ucsb.edu/~parhami/pubs_folder/parh99-asilo-table-sq-root.pdf)
- [Timing square root](https://web.archive.org/web/20210208132927/http://assemblyrequired.crashworks.org/timing-square-root/)

## 参考

- [linux2024-quiz3](https://hackmd.io/@sysprog/linux2024-quiz3)
- [linux2024-quiz2](https://hackmd.io/@sysprog/linux2024-quiz2)
- [gcc:__builtin_clz](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html)
- [Digit by digit calculation](https://en.wikipedia.org/wiki/Methods_of_computing_square_roots#Digit-by-digit_calculation)
