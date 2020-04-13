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

## 环境

```bash
C:\>ver 
MS-DOS Version 6.22  
C:\MASM>masm
Microsoft (R) MASM Compatibility Driver Version 6.11  
Copyright (C) Microsoft Corp 1993.  A1l rights reserved.
```

## 实验1 查看CPU和内存，用机器指令和汇编指令编程  

1. 使用Debug，将下面的程序段写入内存，逐条执行，观察每条指令执行后，CPU中相关寄存器的变化。  
    提示，可用E命令和A命令以两种方式将指令写入内存。注意用T命令执行时，CS:IP的指向。  
    注：这里略去其他寄存器的变化
    ```x86asm
    mov ax,4e20h    ;ax = 4e20h
    add ax,1416h    ;ax = 6236h
    mov bx,2000h    ;bx = 2000h
    add ax,bx       ;ax = 8236h
    mov bx,ax       ;bx = 8236h
    add ax,bx       ;ax = 046ch
    mov ax,001ah    ;ax = 001ah
    mov bx,0026h    ;bx = 0026h
    add al,bl       ;ax = 0040h
    add ah,bl       ;ax = 2640h
    add bh,al       ;bx = 4026h
    mov ah,0        ;ax = 0040h
    add al,bl       ;ax = 0066h
    add al,9ch      ;ax = 0002h
    ```
    
2. 将下面3条指令写入从2000:0开始的内存单元中，利用这3条指令计算2的8次方。  
    ```x86asm
    ;2^8 = 256 = 100h
    mov ax, 1
    add ax, ax
    jmp 2000:0003
    ```
    每次执行 add ax,ax 相当于将ax乘2，重复执行该条指令8次即可。

3. 查看内存中的内容。  
    PC机主板上的ROM写有一个生产日期，在内存FFF00H～FFFFFH的某几个单元中，请找到这个生产日期并试图改变它。  
    提示，如果读者对实验的结果感到疑惑，请仔细阅读第1章中的1.15节。  
    结果如图：04/13/18，只读不可改写  
    ![LAB1_3 Result](http://q7qon7hdm.bkt.clouddn.com/images/asmLab/lab1_3.jpg)

4. 向内存从B8100H开始的单元中填写数据，如：  
    ```bash
    -e B810:0000 01 01 02 02 03 03 04 04
    ```
    请读者先填写不同的数据，观察产生的现象；再改变填写的地址，观察产生的现象。  
    提示，如果读者对实验的结果感到疑惑，请仔细阅读第1章中的1.15节。  
    注：显存区域，结果略

## 实验2 用机器指令和汇编指令编程

1. 使用Debug，将下面的程序段写入内存，逐条执行，根据指令执行后的实验运行情况填空。  
    注：结果不唯一
    ```x86asm
    mov ax,ffff
    mov ds,ax

    mov ax,2200
    mov ss,ax

    mov sp,0100

    mov ax,[0]  ;ax = 5BEA
    add ax,[2]  ;ax = 5CCA
    mov bx,[4]  ;bx = 30F0
    add bx,[6]  ;bx = 6024

    push ax     ;sp = 00FE; 修改的内存单元的地址是 220FE 内容为 5CCA
    push bx     ;sp = 00FC; 修改的内存单元的地址是 220FC 内容为 6024
    pop ax      ;sp = 00FE; ax = 6024
    pop bx      ;sp = 0100; bx = 5CCA

    push [4]    ;sp = 00FE; 修改的内存单元的地址是 220FE 内容为 30F0
    push [6]    ;sp = 00FC; 修改的内存单元的地址是 220FC 内容为 2F34
    ```
2. 仔细观察图3.19中的实验过程，然后分析：为什么2000:0～2000:f中的内容会发生改变？  
    ![IMG 3.19](http://q7qon7hdm.bkt.clouddn.com/images/asmLab/img3.19.jpg)  
    在使用T命令进行单步追踪的时候，产生了中断，CS和IP等寄存器依此入栈，导致了内存相关位置内容的改变。

## 实验3 编程、编译、连接、跟踪

1. 将下面的程序保存为 t1.asm 文件，将其生成可执行文件 t1.exe。
    ```x86asm
    assume cs:codeseg

    codeseg segment
            mov ax,2000H
            mov ss,ax
            mov sp,0
            add sp,10
            pop ax
            pop bx
            push ax
            push bx
            pop ax
            pop bx

            mov ax,4c00H
            int 21H

    codeseg ends

    end
    ```

2. 用Debug跟踪 t1.exe 的执行过程，写出每一步执行后，相关寄存器中的内容和栈顶的内容。  
    结果略
3. PSP的头两个字节是CD 20，用Debug加载 t1.exe，查看PSP的内容。  
    注：这里应该是"-u 0A67:0"  
        即 0A67 = CS-0010h  
    ![PSP](http://q7qon7hdm.bkt.clouddn.com/images/asmLab/lab3_result_u_ds.jpg)

## 实验4 [bx]和loop的使用

1. 编程，向内存0:200～0:23F依次传送数据0～63(3FH)。  
2. 编程，向内存0:200～0:23F依此传送数据0～63(3FH)，程序中只能使用9条指令，9条指令中包括"mov ax,4c00h"和"int 21h"。  
    ```x86asm
    assume cs:code

    code segment
        mov ax,0020h
        mov ds,ax
        mov bx,0

        mov cx,40h
        s:
        mov [bx],bx
        inc bx
        loop s

        mov ax,4c00h
        int 21h
    code ends

    end
    ```
3. 下面的程序的功能是将"mov ax, 4c00h"之前的指令复制到内存的0:200处，补全程序，上机调试，跟踪运行结果。  
    ```x86asm
    assume cs:code

    code segment
        mov ax,cs
        mov ds,ax

        mov ax,0020h
        mov es,ax

        mov bx,0
        mov cx,0017h
        s:
        mov al,[bx]
        mov es:[bx],al
        inc bx
        loop s

        mov ax,4c00h
        int 21h
    code ends

    end
    ```
    提示：  
    - 复制的是什么？从哪里到哪里？  
        复制的是此程序中，从CS:0到"mov ax,4c00h"首地址之间的代码，以字节为单位
    - 复制的是什么？有多少个字节？你如何知道要复制的字节的数量？  
        cs:0--cs:17h = 18h = 24 个字节  
        注意：地址是从0开始的  
        Debug中使用"u cs:0"观察得出  
        参考：可以使用offset计算

## 实验5 编写、调试具有多个段的程序

1. 将下面的程序编译、连接，用Debug加载、跟踪，然后回答问题。
    ```x86asm
    assume cs:code,ds:data,ss:stack

    data segment
        dw 0123h,0456h,0789h,0abch,0defh,0fedh,0cbah,0987h
    data ends

    stack segment
        dw 0,0,0,0,0,0,0,0
    stack ends

    code segment
    start:
        mov ax,stack
        mov ss,ax
        mov sp,10h

        mov ax,data
        mov ds,ax

        push ds:[0]
        push ds:[2]
        pop ds:[2]
        pop ds:[0]

        mov ax,4c00h
        int 21h
    code ends

    end start
    ```
    - CPU执行程序，程序返回前，data段中的数据为多少？  
        data段中的数据不变  
        ```
        -d ds:0 f
        0A77:0000 23 01 56 04 89 07 BC 0A-EF 0D ED 0F BA 0C 87 09
        ```
    - CPU执行程序，程序返回前，CS = 0A79 、SS = 0A78 、DS = 0A77。(答案不唯一)
    - 设程序加载后，code段的段地址为X，则data段的段地址为 X-2 ，stack段的段地址为 X-1 。

2. 将下面的程序编译、连接，用Debug加载、跟踪，然后回答问题。  
    ```x86asm
    assume cs:code,ds:data,ss:stack

    data segment
        dw 0123h,0456h
    data ends

    stack segment
        dw 0,0
    stack ends

    code segment
    start:
        mov ax,stack
        mov ss,ax
        mov sp,10h

        mov ax,data
        mov ds,ax

        push ds:[0]
        push ds:[2]

        pop ds:[2]
        pop ds:[0]

        mov ax,4c00h
        int 21h
    code ends

    end start
    ```
    - CPU执行程序，程序返回前，data段中的数据为多少？  
        data段中的数据不变
        ```
        -d ds:0 f
        0A77:0000 23 01 56 04 00 00 00 00-00 00 00 00 00 00 00 00
        ```
    - CPU执行程序，程序返回前，CS = 0A79 、SS = 0A78 、DS = 0A77 。(答案不唯一)
    - 设程序加载后，code段的段地址为X，则data段的段地址为 X-2 ，stack段的段地址为 X-1 。
    - 对于如下定义的段：
        ```x86asm
        name segment
            ...
        name ends
        ```
        如果段中的数据占N个字节，则程序加载后，该段实际占有的空间为 ((N-1)/16+1)*16 字节。  
        注："(N-1)/16+1"为整除,即"N/16"并向上取整：以16字节(Paragraph)对齐

3. 将下面的程序编译、连接，用Debug加载、跟踪，然后回答问题。  
    ```x86asm
    assume cs:code,ds:data,ss:stack

    code segment
    start:mov ax,stack
        mov ss,ax
        mov sp,10h

        mov ax,data
        mov ds,ax

        push ds:[0]
        push ds:[2]
        pop ds:[2]
        pop ds:[0]

        mov ax,4c00h
        int 21h
    code ends

    data segment
        dw 0123h,0456h
    data ends

    stack segment
        dw 0,0
    stack ends

    end start
    ```
    - CPU执行程序，程序返回前，data段中的数据为多少？  
        data段中的数据不变
        ```
        -d ds:0 f
        0A7A:0000 23 01 56 04 00 00 00 00-00 00 00 00 00 00 00 00
        ```
    - CPU执行程序，程序返回前，CS = 0A77 、SS = 0A7B 、DS = 0A7A 。(答案不唯一)
    - 设程序加载后，code段的段地址为X，则data段的段地址为 X+3 ，stack段的段地址为 X+4 。

4. 如果将1、2、3题中的最后一条伪指令"end start"改为"end"（也就是说，不指明程序的入口），则哪个程序仍然可以正确执行？请说明原因。  
    只有题3中程序可以正确运行，在不指明程序入口的情况下，程序默认按照顺序从头开始执行。  
    而3个程序中只有题3中程序的code段位于最开始的部分，所以只有题3中程序可以正确运行。  

5. 程序如下，编写code段中的代码，将a段和b段中的数据依此相加，将结果存到c段中。  
    ```x86asm
    assume cs:code
    a segment
        db 1,2,3,4,5,6,7,8
    a ends

    b segment
        db 1,2,3,4,5,6,7,8
    b ends

    c segment
        db 0,0,0,0,0,0,0,0
    c ends

    code segment
    start:
            ?
    code ends

    end start
    ```
    注：在博主的[DOS环境与MASM版本](#toc-heading-1)下，"c"是保留关键字，故改写为"cc segment"
    ```x86asm
    assume cs:code

    a segment
    db 1,2,3,4,5,6,7,8
    a ends

    b segment
    db 1,2,3,4,5,6,7,8
    b ends
    
    cc segment
    db 0,0,0,0,0,0,0,0 
    cc ends

    code segment
    start:mov ax,cc
        mov ds,ax

        mov bx,0
        mov cx,8
        s:
        mov dl,0
        mov ax,a
        mov es,ax
        add dl,es:[bx]
        mov ax,b
        mov es,ax
        add dl,es:[bx]
        mov [bx],dl
        inc bx
        loop s

        mov ax,4c00h
        int 21h
    code ends

    end start
    ```

6. 程序如下，编写code段中的代码，用push指令将a段中的前8个**字型**数据，逆序存储到b段中。  
    ```x86asm
    assume cs:code

    a segment
        dw 1,2,3,4,5,6,7,8,9,0ah,0bh,0ch,0dh,0eh,0fh,0ffh
    a ends

    b segment
        dw 0,0,0,0,0,0,0,0
    b ends

    code segment
    start:  
            ?
    code ends

    end start
    ```
    注：进行栈操作时，注意数据的大小
    ```x86asm
    assume cs:code

    a segment
    dw 1,2,3,4,5,6,7,8,9,0ah,0bh,0ch,0dh,0eh,0fh,0ffh
    a ends

    b segment
    dw 0,0,0,0,0,0,0,0
    b ends

    code segment
    start:
        mov ax,a
        mov ds,ax

        mov ax,b
        mov ss,ax
        mov sp,10h

        mov cx,8
        mov bx,0

        s:
        push [bx]
        add bx,2
        loop s

        mov ax,4c00h
        int 21h
    code ends

    end start
    ```

## 实验6 实践课程中的程序

- 未完