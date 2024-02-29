---
title: "Colors of Balls Problem"
date: 2022-04-20T13:37:58+08:00
draft: false
tags: ["math"]
slug: "Colors_of_balls_problem"
mathjax: true
---

n balls with N different colors, number of balls with each color is same, randomly choose k balls from the n balls, the expected number of colors of the k balls can be estimated as:

$$
N\left [ 1-\prod_{r=1}^{k} \left ( \frac{\frac{n(N-1)}{N}-r+1}{n-r+1} \right )  \right ]
$$

## 证明

要算取出的k个球的颜色数的期望, 根据期望的定义来算的话太复杂了.

因此, 可以先构造一个0-1分布, 取出的球中不包含一种颜色的情况用0表示, 则包含这种颜色的情况用1表示.

可以轻易的求出取出的球中不包含一种颜色的概率为:

$$
P_{0}=\frac{C_{n - \frac{n}{N}}^{k}}{C_{n}^{k}}=\frac{(n-\frac{n}{N})(n-\frac{n}{N}-1)\cdots(n-\frac{n}{N}-k+1)}{n(n-1)\cdots(n-k+1)}=\prod_{r=1}^{k}\frac{\frac{n(N-1)}{N}-r+1}{n-r+1}
$$

易得$P_1=1-P_0$, 则该事件的期望为:

$$
E_{0-1}=0\times P_0 + 1\times P_1=1-P_0=1-\prod_{r=1}^{k}\frac{\frac{n(N-1)}{N}-r+1}{n-r+1}
$$

而对于取出k个球中颜色数的数学期望, 可以先对N个不同的颜色编号$1\cdots N$, 利用上面的0-1分布, 设1号颜色的0-1分布为{0, 1}, 2号颜色的0-1分布为{0, 1}, ..., N号颜色的0-1分布为{0, 1}(彼此为相互独立事件, 非互斥事件).

将1号和2号两个事件叠加可得叠加后的分布情况为{0, 1, 2}, 0表示不包含这两种颜色的情况, 1表示包含这两种颜色中的一种的情况, 2表示同时包含这两种颜色的情况. 因此, 依次将这N个事件叠加, 对其求期望及为所求.

根据$E(X+Y)=E(X)+E(Y)$得:

$$
E=E_{0-1}(1)+E_{0-1}(2)+\cdots +E_{0-1}(N)=N(1-P_0)=N\left [ 1-\prod_{r=1}^{k} \left ( \frac{\frac{n(N-1)}{N}-r+1}{n-r+1} \right )  \right ]
$$

证毕.
