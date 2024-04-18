---
title: "C 语言函数调用和VLA和alloca栈的变化探讨"
date: 2024-04-17T11:36:45+08:00
tags: ["c"]
categories: ""
author: "Jiao shijie"
draft: true
hidemeta: true
math: false
---

**NOTE**: 以下讨论使用的平台为 x86-64，使用的编译器为 gcc，以下提供的伪汇编码
采用 intel 格式。

文章中提到的代码，可以在 [Code](https://github.com/jiaoshijie/code_misc/tree/main/c-like/function_call_VLA_alloca) 找到。

## 一些重要的寄存器和指令

函数调用过程中，比较重要的寄存器主要有三个 `rip` `rbp` `rsp`。

- `rip` 寄存器存放的为下一条要执行指令的地址，Instruction Pointer。
- `rbp` 为基寄存器，存放的为前一个 `rbp` 的值，Base Pointer。
- `rsp` 为栈寄存器，一直指向函数调用栈的底部，Stack Pointer。

比较重要的指令有 `call` `push` `pop` `leave` `ret`。

`call addr` 指令会首先将 call 返回之后要执行的地址压入栈中(返回地址)，设置 `rip`
的值为 addr，然后跳转的这个位置去执行，大致可以等效于一下指令。

```asm
# 假设当执行到 call 指令时 CPU 就会自动设置 rip 寄存器为下一条要执行的指令
push rip
mov rip, addr
# 按理设置了 rip 寄存器，CPU 就会去执行 rip 地址的指令，这里写 jmp 只是想要更清晰的表示 call 的流程
jmp addr
```

`push rbp` 指令会将 rsp 寄存器下移一个 word 的长度，rbp 中的内容移动到 rsp
寄存器所指向的地址中。

```asm
sub rsp, 0x8
mov QWORD PTR [rsp], rbp
```

`pop rbp` 指令会将当前 rsp 寄存器所指向地址中的内容移动到 rbp 寄存器中，并将 rsp
寄存器上移一个 word 的长度。

```asm
mov rbp, QWORD PTR [rsp]
add rsp, 0x8
```

`leave` 指令会将 rsp 寄存器指向 rbp 当前指向的位置(这个位置存储的为前一个 rbp
的内容)，然后将 rbp 寄存器的内容设置为 前一个 rbp 的位置。

```asm
mov rsp, rbp
pop rbp
```

`ret` 指令会将返回地址设置到 rip 寄存器中，然后跳转到该位置执行。

```asm
pop rip
jmp rip
```

## 实例

```c
// file name: call.c
int func1(int a, int b, int *c) {
  return a + b;
}

void func2(int *c) {
  int a = 99;
  int b = 1;
  int d = func1(a, b, c);
  a = a + d;
}

int main() {
  int *c = 0;
  func2(c);
  return 0;
}
```

使用 `gcc -g -no-pie -o call call.c` 编译以上代码。然后使用 `gdb -q call` 调试。

进入 `gdb` 后首先，`set disassembly-flavor intel` 将 `disassemble` 命令转换出
的汇编码设置成 intel 格式，当然如果熟悉 AT&T 汇编格式可以不执行这个命令。

然后 `b main` 在 main 函数处设置断点。使用 `disassemble main` 可以查看 main 函数
的汇编码。

```asm
   0x0000000000401202 <+0>:     endbr64
   0x0000000000401206 <+4>:     push   rbp
   0x0000000000401207 <+5>:     mov    rbp,rsp
   0x000000000040120a <+8>:     sub    rsp,0x10
=> 0x000000000040120e <+12>:    mov    QWORD PTR [rbp-0x8],0x0
   0x0000000000401216 <+20>:    mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040121a <+24>:    mov    rdi,rax
   0x000000000040121d <+27>:    call   0x401122 <func2>
   0x0000000000401222 <+32>:    mov    eax,0x0
   0x0000000000401227 <+37>:    leave
   0x0000000000401228 <+38>:    ret
```

可以使用 `p/x $rbp` `p/x $rsp` `p/x $pc` 寄存器中的的内容。

```bash
(gdb) p/x $rbp
$5 = 0x7fffffffe290
(gdb) p/x $rsp
$6 = 0x7fffffffe280
(gdb) p/x $pc
$7 = 0x40120e
```

在 `call` 命令之前的 `mov rdi,rax` 命令为函数的参数传递。函数的参数传递不同的
类型所使用的寄存器也有所不同，如 integer 类型，使用的寄存器依次为 `rdi` `rsi`
`rdx` `rcx` `r8` `r9`，如果 integer 参数多于可以使用的寄存器个数会使用栈来进行
参数传递，对于不同类型所使用的寄存器这个 [PDF](http://www.uclibc.org/docs/psABI-x86_64.pdf) 有详细的介绍。

使用 `si` 来进行指令级别的单步进入调试。当执行完 `call` 命令后再次查看三个寄存器
的内容。

```bash
(gdb) p/x $rbp
$8 = 0x7fffffffe290
(gdb) p/x $rsp
$9 = 0x7fffffffe278
(gdb) p/x $pc
$10 = 0x401122
```

使用 `disassemble func2` 来查看 func2 函数的汇编码。

```asm
=> 0x0000000000401122 <+0>:     endbr64
   0x0000000000401126 <+4>:     push   rbp
   0x0000000000401127 <+5>:     mov    rbp,rsp
   0x000000000040112a <+8>:     sub    rsp,0x18
   0x000000000040112e <+12>:    mov    QWORD PTR [rbp-0x18],rdi
   0x0000000000401132 <+16>:    mov    DWORD PTR [rbp-0xc],0x63
   0x0000000000401139 <+23>:    mov    DWORD PTR [rbp-0x8],0x1
   0x0000000000401140 <+30>:    mov    rdx,QWORD PTR [rbp-0x18]
   0x0000000000401144 <+34>:    mov    ecx,DWORD PTR [rbp-0x8]
   0x0000000000401147 <+37>:    mov    eax,DWORD PTR [rbp-0xc]
   0x000000000040114a <+40>:    mov    esi,ecx
   0x000000000040114c <+42>:    mov    edi,eax
   0x000000000040114e <+44>:    call   0x401106 <func1>
   0x0000000000401153 <+49>:    mov    DWORD PTR [rbp-0x4],eax
   0x0000000000401156 <+52>:    mov    eax,DWORD PTR [rbp-0x4]
   0x0000000000401159 <+55>:    add    DWORD PTR [rbp-0xc],eax
   0x000000000040115c <+58>:    nop
   0x000000000040115d <+59>:    leave
   0x000000000040115e <+60>:    ret
```

`p/x *(long *)$rsp` 可以查看当前 rsp 寄存器所指向的位置的内容 `0x401222`。

可以看到这个地址就是 main 函数中 `call` 指令之后的那条指令的地址，也就是返回地址。

然后使用 `si` 继续执行，当执行完 `push rbp` 指令后，继续查看这三个寄存器中的内容。

```bash
(gdb) p/x $rbp
$12 = 0x7fffffffe290
(gdb) p/x $rsp
$13 = 0x7fffffffe270
(gdb) p/x $pc
$14 = 0x401127
```

同样使用 `p/x *(long *)$rsp` 查看 rsp 寄存器所指向的位置中的内容 `0x7fffffffe290`。
这个值就是现在 rbp 寄存器中的内容，而下一条指令 `mov rbp,rsp` 就是将 rbp 指向
当前 rsp 所指向的位置。

```bash
(gdb) p/x $rbp
$17 = 0x7fffffffe270
(gdb) p/x $rsp
$18 = 0x7fffffffe270
```

下面的一部分命令就是将函数的参数和局部变量放入栈中。

接下来，直接跳转到 func2 函数中的 `leave` 指令位置，当执行完 `leave` 指令后再次
查看三个寄存器中的值。

```bash
(gdb) p/x $rbp
$17 = 0x7fffffffe290
(gdb) p/x $rsp
$18 = 0x7fffffffe278
(gdb) p/x $pc
$19 = 0x40115e
```

可以看到 rbp 寄存器又指向了在 func2 函数调用之前所指向的位置，rsp 寄存器目前指向
的为返回地址的位置。

然后执行 `ret` 指令后，三个寄存器全部变为调用 func2 之前的状态，对 func2 的函数
调用过程执行完成。

```bash
(gdb) p/x $rbp
$21 = 0x7fffffffe290
(gdb) p/x $rsp
$22 = 0x7fffffffe280
(gdb) p/x $pc
$23 = 0x401222
```

上述过程的图示。

![function call stack layout](../../images/function_call_VLA_alloca_01.png#center)

## Variable Length Array(VLA) 和 alloca

在 ISO C99 之后 C 语言支持 variable length array，就是使用变量作为数组的大小。
而这个数据依然是放在 stack 当中，alloca 的实现方式和 VLA 基本相同放在一起探讨。

使用下面的 C 代码来进行演示

```c
// file name: vla.c
int func1(int n) {
  int b = 0xbe;
  int a[n];
  int c = 0xef;
  a[n - 1] = n * 2;
  a[n - 1] += 2;
  return a[n - 1];
}

int main() {
  int a = func1(10);

  a = func1(100);

  return 0;
}
```

同样使用 `gcc -g -no-pie -o vla vla.c`，然后使用 `gdb -q vla` 进行调试，
使用 `disassemble func1` 可以得到它的如下汇编码

```asm
0x0000000000401136 <+0>:     endbr64
0x000000000040113a <+4>:     push   rbp
0x000000000040113b <+5>:     mov    rbp,rsp
0x000000000040113e <+8>:     push   rbx
0x000000000040113f <+9>:     sub    rsp,0x38
0x0000000000401143 <+13>:    mov    DWORD PTR [rbp-0x34],edi
0x0000000000401146 <+16>:    mov    rax,QWORD PTR fs:0x28
0x000000000040114f <+25>:    mov    QWORD PTR [rbp-0x18],rax
0x0000000000401153 <+29>:    xor    eax,eax
0x0000000000401155 <+31>:    mov    rax,rsp
0x0000000000401158 <+34>:    mov    rsi,rax
0x000000000040115b <+37>:    mov    DWORD PTR [rbp-0x30],0xbe
0x0000000000401162 <+44>:    mov    eax,DWORD PTR [rbp-0x34]
0x0000000000401165 <+47>:    movsxd rdx,eax
0x0000000000401168 <+50>:    sub    rdx,0x1
0x000000000040116c <+54>:    mov    QWORD PTR [rbp-0x28],rdx
0x0000000000401170 <+58>:    movsxd rdx,eax
0x0000000000401173 <+61>:    mov    r8,rdx
0x0000000000401176 <+64>:    mov    r9d,0x0
0x000000000040117c <+70>:    movsxd rdx,eax
0x000000000040117f <+73>:    mov    rcx,rdx
0x0000000000401182 <+76>:    mov    ebx,0x0
0x0000000000401187 <+81>:    cdqe
0x0000000000401189 <+83>:    lea    rdx,[rax*4+0x0]
0x0000000000401191 <+91>:    mov    eax,0x10
0x0000000000401196 <+96>:    sub    rax,0x1
0x000000000040119a <+100>:   add    rax,rdx
0x000000000040119d <+103>:   mov    ebx,0x10
0x00000000004011a2 <+108>:   mov    edx,0x0
0x00000000004011a7 <+113>:   div    rbx
0x00000000004011aa <+116>:   imul   rax,rax,0x10
0x00000000004011ae <+120>:   mov    rcx,rax
0x00000000004011b1 <+123>:   and    rcx,0xfffffffffffff000
0x00000000004011b8 <+130>:   mov    rdx,rsp
0x00000000004011bb <+133>:   sub    rdx,rcx
0x00000000004011be <+136>:   cmp    rsp,rdx
0x00000000004011c1 <+139>:   je     0x4011d5 <func1+159>
0x00000000004011c3 <+141>:   sub    rsp,0x1000
0x00000000004011ca <+148>:   or     QWORD PTR [rsp+0xff8],0x0
0x00000000004011d3 <+157>:   jmp    0x4011be <func1+136>
0x00000000004011d5 <+159>:   mov    rdx,rax
0x00000000004011d8 <+162>:   and    edx,0xfff
0x00000000004011de <+168>:   sub    rsp,rdx
0x00000000004011e1 <+171>:   mov    rdx,rax
0x00000000004011e4 <+174>:   and    edx,0xfff
0x00000000004011ea <+180>:   test   rdx,rdx
0x00000000004011ed <+183>:   je     0x4011ff <func1+201>
0x00000000004011ef <+185>:   and    eax,0xfff
0x00000000004011f4 <+190>:   sub    rax,0x8
0x00000000004011f8 <+194>:   add    rax,rsp
0x00000000004011fb <+197>:   or     QWORD PTR [rax],0x0
0x00000000004011ff <+201>:   mov    rax,rsp
0x0000000000401202 <+204>:   add    rax,0x3
0x0000000000401206 <+208>:   shr    rax,0x2
0x000000000040120a <+212>:   shl    rax,0x2
0x000000000040120e <+216>:   mov    QWORD PTR [rbp-0x20],rax
0x0000000000401212 <+220>:   mov    DWORD PTR [rbp-0x2c],0xef
0x0000000000401219 <+227>:   mov    eax,DWORD PTR [rbp-0x34]
0x000000000040121c <+230>:   lea    edx,[rax-0x1]
0x000000000040121f <+233>:   mov    eax,DWORD PTR [rbp-0x34]
0x0000000000401222 <+236>:   lea    ecx,[rax+rax*1]
0x0000000000401225 <+239>:   mov    rax,QWORD PTR [rbp-0x20]
0x0000000000401229 <+243>:   movsxd rdx,edx
0x000000000040122c <+246>:   mov    DWORD PTR [rax+rdx*4],ecx
0x000000000040122f <+249>:   mov    eax,DWORD PTR [rbp-0x34]
0x0000000000401232 <+252>:   lea    edx,[rax-0x1]
0x0000000000401235 <+255>:   mov    rax,QWORD PTR [rbp-0x20]
0x0000000000401239 <+259>:   movsxd rdx,edx
0x000000000040123c <+262>:   mov    eax,DWORD PTR [rax+rdx*4]
0x000000000040123f <+265>:   mov    edx,DWORD PTR [rbp-0x34]
0x0000000000401242 <+268>:   sub    edx,0x1
0x0000000000401245 <+271>:   lea    ecx,[rax+0x2]
0x0000000000401248 <+274>:   mov    rax,QWORD PTR [rbp-0x20]
0x000000000040124c <+278>:   movsxd rdx,edx
0x000000000040124f <+281>:   mov    DWORD PTR [rax+rdx*4],ecx
0x0000000000401252 <+284>:   mov    eax,DWORD PTR [rbp-0x34]
0x0000000000401255 <+287>:   lea    edx,[rax-0x1]
0x0000000000401258 <+290>:   mov    rax,QWORD PTR [rbp-0x20]
0x000000000040125c <+294>:   movsxd rdx,edx
0x000000000040125f <+297>:   mov    eax,DWORD PTR [rax+rdx*4]
0x0000000000401262 <+300>:   mov    rsp,rsi
0x0000000000401265 <+303>:   mov    rdx,QWORD PTR [rbp-0x18]
0x0000000000401269 <+307>:   sub    rdx,QWORD PTR fs:0x28
0x0000000000401272 <+316>:   je     0x401279 <func1+323>
0x0000000000401274 <+318>:   call   0x401040 <__stack_chk_fail@plt>
0x0000000000401279 <+323>:   mov    rbx,QWORD PTR [rbp-0x8]
0x000000000040127d <+327>:   leave
0x000000000040127e <+328>:   ret
```

![vla and alloca](../../images/function_call_VLA_alloca_02.png#center)

```asm
0x0000000000401146 <+16>:    mov    rax,QWORD PTR fs:0x28
0x000000000040114f <+25>:    mov    QWORD PTR [rbp-0x18],rax
...
0x0000000000401265 <+303>:   mov    rdx,QWORD PTR [rbp-0x18]
0x0000000000401269 <+307>:   sub    rdx,QWORD PTR fs:0x28
0x0000000000401272 <+316>:   je     0x401279 <func1+323>
0x0000000000401274 <+318>:   call   0x401040 <__stack_chk_fail@plt>
0x0000000000401279 <+323>:   mov    rbx,QWORD PTR [rbp-0x8]
```

这两条指令就是设置图片中的 stack guard，用于检查对栈的操作是否越界了。可以看到
在这个函数的最后会检查这个位置的值是否和原始相同，如果相同就会跳转到 0x401279处
执行，否则就会执行 __stack_chk_fail 函数，产生 segmentation falut 错误。这个检查
主要是为了防止修改掉 prev rbp 和返回地址从而导致程序执行错误。

```asm
0x0000000000401153 <+29>:    xor    eax,eax
0x0000000000401155 <+31>:    mov    rax,rsp
0x0000000000401158 <+34>:    mov    rsi,rax
```

设置 rsi 暂存 rsp 当前的值。

从 0x401162 到 0x40120e 为计算和设置 VLA 的开始地址，即图中的蓝色部分，同时也将
rsp 设置到栈底。

从 0x401219 到 0x40124f 对应的为 C 代码中的对 `a[n-1]` 赋值的两行。

从 0x401252 到 0x40125f 对应的为 C 代码中的 `return a[n-1]`。

```
0x0000000000401262 <+300>:   mov    rsp,rsi
```

恢复 rsp 寄存器为 0x7fffffffe230。

然后正常退出函数。

为什么在函数开头和结尾 `push rbx` and `mov    rbx,QWORD PTR [rbp-0x8]`?

![vla and alloca](../../images/function_call_VLA_alloca_03.png#center)

根据上图可以看到被调用的函数如果有用到 rbx 寄存器，是有责任保存这个寄存器的。


## Ref

- [what registers are preserved through a linux x86_64 function call](https://stackoverflow.com/questions/18024672/what-registers-are-preserved-through-a-linux-x86-64-function-call)
- [what does the endbr64 instruction actually do](https://stackoverflow.com/questions/56905811/what-does-the-endbr64-instruction-actually-do)
- [gcc VLA](https://gcc.gnu.org/onlinedocs/gcc/Variable-Length.html)
- [x86 and amd64 instruction reference](https://www.felixcloutier.com/x86/)
- [x86 what does movsxd rdx edx instruction mean](https://stackoverflow.com/questions/56565510/x86-what-does-movsxd-rdx-edx-instruction-mean)
- [x86-64 reference sheet.pdf](https://people.kth.se/~dbro/x86-64-ref-sheet.pdf)
- [difference between je jne and jz jnz](https://stackoverflow.com/questions/14267081/difference-between-je-jne-and-jz-jnz)
- [why does this memory address fs0x28 fs0x28 have a random value](https://stackoverflow.com/questions/10325713/why-does-this-memory-address-fs0x28-fs0x28-have-a-random-value)
