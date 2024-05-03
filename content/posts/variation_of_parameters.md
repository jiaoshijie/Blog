---
title: "常数变易法(微分方程求解)"
date: 2024-05-03T19:06:23+08:00
tags: ["math"]
categories: ""
author: "Jiao shijie"
draft: false
hidemeta: true
math: true
---

## 1. 常数变易法求解一阶线性微分方程过程

对于一阶线性微分方程:

$$
\frac{\mathrm{d}y}{\mathrm{d}x}+P(x)y=Q(x) \tag{1}
$$

如果$Q(x)=0$则方程为齐次, $Q(x)\ne0$则方程为非齐次.

而常数变易法的求解就是先求出齐次方程的通解

$$
\frac{\mathrm{d}y}{\mathrm{d}x}+P(x)y=0 \tag{2}
$$

该齐次方程的通解为$y=Ce^{-\int{P(x)\mathrm{d}x}}$ $(C=\pm e^{C_1})$

然后使用$u(x)$(以后简写为$u$)替换该通解中的常数$C$, 即便换为:

$$
y=ue^{-\int{P(x)\mathrm{d}x}} \tag{3}
$$

然后, 将(3)代入(1)中, 得:

$$
u^{\prime}e^{-\int{P(x)}\mathrm{d}x}=Q(x) \tag{4}
$$

通过"分离变量"即可求出$u$:

$$
u=\int{Q(x)e^{\int{P(x)\mathrm{d}x}}\mathrm{d}x}+C \tag{5}
$$

将(5)在带回到(3)中即可求出$y$:

$$
y=Ce^{-\int{P(x)\mathrm{d}x}}+e^{-\int{P(x)\mathrm{d}x}}\int{Q(x)e^{\int{P(x)\mathrm{d}x}}\mathrm{d}x}
$$

于是, 便得到了(1)的通解, 观察可以发现, 前面为(1)对应的齐次方程的通解, 后面为当C=0是(1)的特解, 因此得到结论:

> 一阶非齐次线性方程的通解等于对应的齐次方程的通解与非齐次方程的一个特解之和.

但是问题就是 **为什么可以使用非齐次方程对应的齐次方程的通解来进行求解?**, 上面的方法一气呵成只是告诉了我们应该这样做, 但我们却不知道为什么这么做.

但我学的很蒙, 于是进行了一番搜索找到了一些好的文章, 打算自己总结一下, 相关文章都放在了*参考*处.

## 2. 原理探讨

通过《"常数变易法"的探讨》文章知道我们现在使用的常数变易法只是结论, 而非推导过程.

> 该结论是拉格朗日十一年的研究成果.

因此在此介绍一些该结论得出的过程.

首先, 对于求解一阶线性微分方程最先想到的方法就是"分离常量", 于是首先对(1)进行移项操作得:

$$
\mathrm{d}y=[Q(x)-P(x)y]\mathrm{d}x
$$

观察怎样都无法将y单独移动到左边, 因此分离常量的方法不可以求解此方程.

于是想到使用$u=\frac{y}{x}$的方法进行求解得到:

$$
u\prime{}x+u(1+P(x)x)=Q(x)
$$

可以看到依然无法将u和x单独分开, 但是可以观察到如果$x=-\frac{1}{P(x)}$则左边第二项就为0, 因此就可以使用分离常数的方法进行求解u了.

因此, 如果要使x是一个函数就可能可以求解处(1)的结果, 而$y=xu$其实就是两个函数的乘积, 只不过$v=x$这个函数函数比较简单不足以使第二项为0, 并且$u=u(x)$本身是不确定的函数.

因此考虑设$v=v(x)$来代替$x$于是

$$
y=v(x)u(x)=vu \tag{6}
$$

因此v和u都是关于x的函数, 而y又是被u和v这两个函数的乘积控制. 因此将(6)代入(1)得到:

$$
u\prime{}v+u(v\prime{}+P(x)v)=Q(x) \tag{7}
$$

因此令$u(v\prime{}+P(x)v)=0$, 因为u不可以为0(不然$y=vu$就没有意义了), 因此即

$$
v\prime{}+P(x)v=0 \tag{8}
$$

而这个式子可以通过分离变量来进行求解, 便可以求出v为:

$$
v=C_1e^{-\int{P(x)\mathrm{d}x}} \tag{9}
$$

将(9)代回到(8)中, 得到:

$$
u\prime{}C_1e^{-\int{P(x)\mathrm{d}x}}=Q(x) \tag{10}
$$

便可以再次通过分离变量来求解出u来:

$$
u=\frac{1}{C_1}\int{Q(x)e^{\int{P(x)\mathrm{d}x}}\mathrm{d}x}+C_2
$$

又因为之前设$y=uv$因此可以得到y为:

$$
y=Ce^{-\int{P(x)\mathrm{d}x}}+e^{-\int{P(x)\mathrm{d}x}}\int{Q(x)e^{\int{P(x)\mathrm{d}x}}\mathrm{d}x}\space(C=C_1C_2)
$$

于是便得到了(1)的通解.

## 3. 为什么可以使用常数变易法替代上述的方法

首先, 我们来观察以下公式(8)如果将因变量v变为因变量y是不是公式(2)也就是对应的齐次方程.

而对于第一部分中对齐次方程求解得到的结果$y=Ce^{-\int{P(x)\mathrm{d}x}}$中将C替换为u, 不就是第二部分中我们设的$y=uv$ 和 $v=e^{-\int{P(x)\mathrm{d}x}}$了吗

因此第一部分便可以继续求解从而得到(1)的通解. 因此第一部分的常数变易法只是第二部分分析求解的简化过程只是正巧可以按常数变易法的方式进行求解, 应该也可以说常数变易法是第二部分观察的结果(结论).

## 参考

- [常数变易法来历的探讨-崔士襄-论文](http://kns.cnki.net/KCMS/detail/detail.aspx?dbcode=CJFQ&dbname=CJFD9899&filename=HDLY199801017&uid=WEEvREcwSlJHSldRa1FhdkJkVWEyZnAyYk9seWN1S2dJbGhMcCtxTERzQT0=$9A4hF_YAuvQ5obgVAqNKPCYcEjKensW4IQMovwHtwkF4VYPoHbKxJw!!&v=MDAwNDFyQ1VSTEtlWitkckZpdm1VYnZLTFNuSGQ3S3hGOW5Ncm85RVk0UjhlWDFMdXhZUzdEaDFUM3FUcldNMUY=)
