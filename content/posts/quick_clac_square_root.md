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

但是如果每次都比较 $P_m^2 \le N$，程序的效率依旧很低，因此我们定义 $X_m$ 为
$N$ 与 $P_m^2$ 差值， 使用 $X_m = X_{m+1} - Y_m$ 来更新 $X_m$，$X_M$ 和 $Y_m$
的定义如下所示

$$
X_m = N - P_m^2 = X_{m+1} - Y_m
$$

$$
Y_m = P_m^2 - P_{m+1}^2 = 2P_{m+1}a_m + a_m^2 = 2^{m+1}P_{m+1} + (2^{m})^2
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

**注意** $c_{-1} = P_02^0 = P_0 = \sqrt{N}$。
