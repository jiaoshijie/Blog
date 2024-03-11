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

其中，$a_i = 2^i$ 或 $a_i = 0$，取决于对应二进制位的值为 1 还是 0。将其展开可以得到

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


