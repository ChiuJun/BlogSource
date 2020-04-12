---
title: Assembly Language Lab
top: false
cover: false
img: 'http://q7qon7hdm.bkt.clouddn.com/medias/featureimages/asm3e.jpg'
toc: true
mathjax: true
date: 2020-04-13 01:32:21
password:
summary: 记录王爽汇编语言第三版实验相关
tags:
    - ASM
    - LAB
categories:
    - 计算机基础
---

# 王爽《汇编语言》第三版 实验

> 以后用到的知识 以后再说  

> 久闻大名，未曾拜读  
    这次趁着微机课程，解去心头之痒  
    顺便博客刚开，就随手记录下书本上的实验  

```bash
C:\>ver 
MS-DOS Version 6.22  
C:\MASM>masm
Microsoft (R) MASM Compatibility Driver Version 6.11  
Copyright (C) Microsoft Corp 1993.  A1l rights reserved.
```

## 实验1 查看CPU和内存，用机器指令和汇编指令编程  

1. 使用Debug，将下面的程序段写入内存，逐条执行，观察每条指令执行后，CPU中相关寄存器的变化。  
    - 提示，可用E命令和A命令以两种方式将指令写入内存。注意用T命令执行时，CS:IP的指向。
    - 注：这里略去其他寄存器的变化
    ```x86asm
    mov ax,4e20h    ;ax=4e20h
    add ax,1416h	;ax=6236h
    mov bx,2000h	;bx=2000h
    add ax,bx	    ;ax=8236h
    mov bx,ax	    ;bx=8236h
    add ax,bx	    ;ax=046ch
    mov ax,001ah    ;ax=001ah
    mov bx,0026h	;bx=0026h
    add al,bl	    ;ax=0040h
    add ah,bl	    ;ax=2640h
    add bh,al	    ;bx=4026h
    mov ah,0	    ;ax=0040h
    add al,bl	    ;ax=0066h
    add al,9ch	    ;ax=0002h
    ```
    
2. 将下面3条指令写入从2000:0开始的内存单元中，利用这3条指令计算2的8次方。
    ```x86asm
    ;2^8=256=100h
    mov ax, 1
    add ax, ax
    jmp 2000:0003
    ```
    每次执行 add ax,ax 相当于将ax乘2，重复执行该条指令8次即可。

3. 查看内存中的内容。  
    PC机主板上的ROM写有一个生产日期，在内存FFF00H～FFFFFH的某几个单元中，请找到这个生产日期并试图改变它。  
    提示，如果读者对实验的结果感到疑惑，请仔细阅读第1章中的1.15节。  
    结果如图：04/13/18，只读不可改写  
    ![LAB1_3 Result](http://q7qon7hdm.bkt.clouddn.com/images/asmLab/lab1_jpg)

4. 向内存从B8100H开始的单元中填写数据，如：
    ```bash
    -e B810:0000 01 01 02 02 03 03 04 04
    ```
    请读者先填写不同的数据，观察产生的现象；再改变填写的地址，观察产生的现象。  
    提示，如果读者对实验的结果感到疑惑，请仔细阅读第1章中的1.15节。  
    注：显存区域，结果略

## 实验2 用机器指令和汇编指令编程

- 睡觉先