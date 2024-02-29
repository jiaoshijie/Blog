---
title: "00 编译器简介"
date: 2022-07-17T14:57:04+08:00
draft: false
tags: ["compiler"]
categories: "compiler"
hidemeta: true
math: false
---

## 1 语言处理器(language processor)

language processor可以分为两种, 一种是编译器(compiler), 一种是解释器(interpreter).

**编译器(compiler)** 是将一种语言转换为另一种语言, 通常是高级语言向low-level语言转换. 在转化过程中如果有错误要向用户报告这个错误. 如果这个low-level语言是可执行的机器语言那它还可以被用户执行.

![Figure: A compiler and running the target program](/images/compiler/00_aCompilerAndRuningTargetProgram.png#center)

**解释器(interpreter)** 是另一类的language processor, 它不会将高级语言编译成low-level语言来执行, 而是直接执行该高级语言.

![Figure: A interpreter](/images/compiler/00_aInterpreter.png#center)

通常来说, 编译型语言要比解释型语言运行快, 但解释型语言可以更好的检查错误, 因为很多runtime error在编译期很难检测出来.

接下来主要以编译器为主进行介绍, 首先看一下编译器在编译一个文件时的流程图.

![Figure: A Language-processing system](/images/compiler/00_aLanguageProcessingSystem.png#center)

根据上图可以看出编译器在将源程序编译目标程序时是分步骤完成的:
1. 预处理(preprocess)
2. 编译(compile)
3. 汇编(assemble)
4. 链接(link)
5. 加载(load)

- **预处理器(preprocessor)** 的作用就是将这些分散的文件聚集起来和将替换文件中的宏(macros), 将结果传递至下一个阶段(编译).
- **编译器(compiler)** 的作用就是将预处理器的结果转化成汇编语言(assembly language), 但是也可以直接转化为机器码, 这样也就可以跳过第3步的汇编阶段了, 但是因为汇编语言也是有语义的语言因此转化起来效率高, 且汇编器的效率也很高, 因此比较典型的编译型语言(C, C++)都是先转成汇编再编译为机器码. 且汇编语言debug也比较简单.
- **汇编器(assembler)** 的作用是将上一步产生的汇编程序转换为可重定位机器码(relocatable machine code).
  > Relocatable code is software whose execution address can be changed.
- **链接器(linker)** 的作用是将产生的各个relocatable machine code文件链接在一起, 确定最终的执行地址.
- **加载器(loader)** 的作用就是将linker生成的执行程序加载到内存当中运行.

## 2 编译器的结构

在上面我们提到编译器就是将一种语言转化为另一种语言. 这是很笼统的一个说法, 如果将编译器在进行细分它又可以分为两部分**解析(analysis)**和**合成(synthesis)**.

- **解析(analysis)** 就是将源程序通过分析产生一种源程序的中间表示(中间程序), 在这个过程种如果发现语法或语义错误要提供错误信息, 以便改错. 而且还会收集源程序的各种信息生成一个**symbol table**和中间程序一起传递给下一个阶段**合成(synthesis)**.
- **合成(synthesis)** 就是利用中间程序和symbol table来产生目标程序.

> 通常将 **解析(analysis)** 称为编译器的前端(front end), 将 **合成(synthesis)** 称为编译器的后端(back end).

如果更加深入地划分编译器, 那么可以被划分为更多的阶段, 下图展示了一个比较经典的划分.

![Figure: Phases of a compiler](/images/compiler/00_phasesOfACompiler.png#center)

### 2.1 Lexical Analysis(scanning)

Lexcial analyzer会把这个程序读进来然后将有意义的词汇组合起来形成*lexemes*. 对于每一个lexemes, lexcial analyzer都会为它产生一个*token*. token的格式为`<token-name, attribute-value>`. 并且会经这些token传递给下一个阶段(syntax analysis).

- *token-name* 是一个抽象的符号, 有点类似与占位符. 比如一个变量会被表示为<identifier>, 就是只要这个位置是个变量就会用<identifier>替换而不是记录变量的名字. 会在syntax analysis阶段用到.
- *attribute-value* 会指向token-name在symbol table中的入口. 会在semantic analysis和code generation阶段用到.

下面通过一个例子来看看lexcial analysis的过程.

```c
position = initial + rate * 60
```

上面这个赋值操作会被lexcial analyzer分析为以下几个部分:

1. `position` 映射为 `<id, 1>`(id是identifier的简写), 
2. `=` 映射为 `<=>`
3. `initial` 映射为 `<id, 2>`
4. `+` 映射为 `<+>`
5. `rate` 映射为 `<id, 3>`
6. `*` 映射为 `<*>`
7. `60` 映射为 `<60>`

以上有一些映射是简化的, 更详细的映射会在后续讲解.

### 2.2 Syntax Analysis(parsing)

parser使用上一阶段生成的tokens来生成一个语法树. 这个树的中间节点表示操作, 孩子节点表示操作的参数.

上面那个赋值操作生成的语法树就像下图这样.

![Figure: Syntax tree](/images/compiler/00_syntaxTree.png#center)

### 2.3 Semantic Analysis

Semantic analyzer使用语法树和symbol table来检查程序的语义是否正确, 并产生一些类型信息(type information)把这些信息存在语法树或symbol table中.

Semantic analysis的一个重要部分就是**类型检查**, 检查操作数是否满足操作符. 比如, 在变成语言中数组的索引需要使用整数如果操作数是一个浮点数那么compiler就会报错.

还有一些语言支持**隐式类型转化(coersion)**, 如果一个+操作, 两个操作数一个是integer, 一个是floating-number, 那么这个integer操作数会被转化为floating-number.

上面那个赋值操作经过Semantic analyzer后就是下图.

![Figure: Semantic analysis](/images/compiler/00_semanticAnalysis.png#center)

`inttofloat` 表示把60转化为floating-number.

### 2.4 Intermediate Code Generation

在syntax analysis和semantic analysis之后, compiler会生成一个low-level或machine-like的中间代码.

这个中间代码必须有两个性质:

1. 可以简单的生成
2. 可以简单的转化成目标语言

接下来会使用`three-address instruction`, 这种表示方法在结构上和汇编比较相似.

下面是赋值操作使用three-address instruction表示的中间代码.

![Figure: Three-address instruction](/images/compiler/00_threeAddressInstuction.png#center)

Three-address instruction也有三个不足的地方:

1. 每一个three-address instruction在=的右侧最多只能有一个操作符, 因此compiler要调整表达式的顺序, 来正确地执行一个运算操作
2. compiler要产生很多的临时名字
3. 有一些three-address instruction只有另个操作数, 就像上图的第一个和最后一个一样.

### 2.5 Code Optimization

代码优化通常指的是使目标程序变得更快, 更小和功耗更低.

下面是对上一阶段的优化的结果.

![Figure: Code optimization](/images/compiler/00_codeOptimzation.png#center)

现代编译器在编译程序时有大量的时间是花费在这个阶段的. 代码优化的原则是能够显著地提高程序的运行效率又不会使编译时间太长. 当然, 最重要的还是目标程序和源程序结果要一致.

### 2.6 Code Generation

在目标代码生成阶段最重要的就是如何明智地分配寄存器(registers)和内存(memory locations)来存储变量.

下面是生成的汇编语言.

![Figure: Code generation](/images/compiler/00_targetCodeGeneration.png#center)

上面生成的代码有一个很重要的问题没有表现出来就是**空间分配(Storage Allocation)**. 空间分配可以在中间代码生成是处理也可以在目标代码生成时处理, 这些将在后面介绍.

接下来看一些编译器的完整流程图.

![Figure: Translation of an assignment statement](/images/compiler/00_translationOfAnAssignmentStatement.png#center)

### 2.7 compiler-construction tools

1. Parsing generators
2. Scanner generators
3. Syntax-directed transaltion engines
4. Code-generator generators
5. Data-flow analysis engines
6. Compiler-construction toolkits

## 3 编程语言的进化

- 编程语言的一个分类: 命令式语言(imperative language)和声明性语言(declarative language)

**命令式语言**考虑的是对于一个问题, 你要如何实现解决问题的方法.

**声明式语言**考虑的是对于一个问题, 你要用什么方法解决这个问题.

举一个例子, 从数据库中读取数据并排序, 用SQL(声明式语言)实现只用写一条SQL语句就可以, 而如果用C(命令式语言)实现需要自己实现读数据的操作和排序操作.

如果还不清楚可以参考[what is declarative language?](https://stackoverflow.com/questions/129628/what-is-declarative-programming)
