---
title: "通过实验验证代码是否为常量时间复杂度"
date: 2024-03-14T10:27:41+08:00
tags: ["math", "c", "algorithm"]
categories: ""
author: "Jiao shijie"
draft: true
hidemeta: true
math: true
---

论文[《Dude, is my code constant time?》](https://eprint.iacr.org/2016/1123.pdf)，
给出了一种dudect的方法，可以实验而非理论的方式验证代码时间复杂度是否为常量时间
($O(1)$)的方法，并且平台无关。

下面以验证对双向链表向头和尾部插入和删除是否为线性时间来探讨该方法的使用方法。

## dudect 算法流程简单介绍

### 1 测量执行时间

该算法会重复测量一个函数在两个不同输入类别的执行时间。

1. 类别定义

类别的选择可以很多样，在原论文及本文的例子中选择的是 fix-vs-random 的类别定义，
fix 类别就是每次执行的的输入都是一个固定的值，random 类别每次执行的的输入会是
一个随机的值。而 fix 类别值的选择通常会选一些特例。

2. CPU周期计数器(cycle counters)

现代的CPU有提供周期计数器，像 x86 家族的 [TSC](https://en.wikipedia.org/wiki/Time_Stamp_Counter)
寄存器，ARM 也有一个可选的 systick peripheral 可用。当然也可以使用一个 high-resolution
timer 来计算时间。在使用 TSC 时也有需要注意的情况，因为每个 CPU core 的 TSC 寄存器
的值是不同的。

3. 环境因素的影响

为了减少环境因素的影响，输入数据和类别的分类要在测试开始前准备好，每次运行的
类别也要是随机的。

### 2 后置处理

1. 裁剪数据

在程序运行时有时可能会因为中断而导致程序测试的时间过长，在这种情况下应该裁剪掉
这些异常数据。或者可以设定和类别无关的一个界限，剔除掉和界限无关的测量结果。

### 3 统计分析

1. t-test
