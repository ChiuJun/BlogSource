---
title: Assembly Language Lab
top: false
cover: false
img: 'http://q7qon7hdm.bkt.clouddn.com/medias/featureimages/asm.jpg'
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

> 以后用到的知识，我们以后再说。

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

## 实验 1 查看CPU和内存，用机器指令和汇编指令编程  

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

## 实验 2 用机器指令和汇编指令编程

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

## 实验 3 编程、编译、连接、跟踪

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

## 实验 4 [bx]和loop的使用

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

## 实验 5 编写、调试具有多个段的程序

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

## 实验 6 实践课程中的程序

1. 将课程中所有讲解过的程序上机调试，用Debug跟踪其执行过程，并在过程中进一步理解所讲内容。  
    略

2. 编程，完成问题7.9中的程序。
    > 问题7.9  
    编程，将 datasg 段中每个单词的前四个字母改为大写字母。  
    ```x86asm
    datasg segment  
        db '1. display      '  
        db '2. brows        '  
        db '3. replace      '  
        db '4. modify       '  
    datasg ends
    ```
    [DOS环境与MASM版本](#toc-heading-1)  
    在编写这个程序的过程中，遇到一个奇怪的问题： 
    ``` x86asm
    ;编译错误 error A2070:invalid instruction
    mov al,[bx+3+si]
    ;编译通过
    mov al,byte ptr [bx+3+si]
    ;编译通过
    mov al,[bx+si+3]
    ;编译通过
    mov ax,[bx+3+si]
    ```
    简单查阅了[MASM6.1文档](https://github.com/ChiuJun/CSAPP_3e_Solutions/blob/master/Reference/MASM6.1%E6%96%87%E6%A1%A3.pdf)，没看出是什么原因导致，以下是完整的程序：
    ```x86asm
    assume cs:codeseg,ss:stackseg,ds:dataseg

    stackseg segment
    dw 0,0,0,0,0,0,0,0
    stackseg ends

    dataseg segment
    db '1. display      '
    db '2. brows        '
    db '3. replace      '
    db '4. modify       '
    dataseg ends

    codeseg segment
    start:
        mov ax,stackseg
        mov ss,ax
        mov sp,10H

        mov ax,dataseg
        mov ds,ax

        mov bx,0
        mov cx,4
    s:
        push cx
        mov si,0
        mov cx,4
        
        i:
        mov al,[bx+si+3]
        and al,11011111B
        mov [bx+si+3],al
        inc si
        loop i

        add bx,10H
        pop cx
    loop s

        mov ax,4c00H
        int 21H
    codeseg ends

    end start
    ```
    ![LAB6_2 Result](http://q7qon7hdm.bkt.clouddn.com/images/asmLab/lab6_2_result.jpg)

## 实验 7 寻址方式在结构化数据访问中的应用

- 编程，将 data 段中的数据按如下格式写入到 table 段中，并计算21年中的人均收入(取整)，结果也按照下面的格式保存在table段中。  
    ![LAB7 格式](http://q7qon7hdm.bkt.clouddn.com/images/asmLab/lab7_format.jpg)  
    编写完之后，博主觉得这样的写法更容易出错，在书上记下了一个简单易懂的想法，之后在[课程设计 1](#toc-heading-12)重新进行了编写。   
    编写这个程序遇到一个新手容易忘记的问题：  
    - mov 指令不能在两个存储单元之间直接传送数据  

    以下是旧版本的代码：
    ```x86asm
    assume cs:codeseg

    data segment
    db '1975','1976','1977','1978','1979','1980','1981','1982','1983'
    db '1984','1985','1986','1987','1988','1989','1990','1991','1992'
    db '1993','1994','1995'

    dd 16,22,382,1356,2390,8000,16000,24486,50065,97479,140417,197514
    dd 345980,590827,803530,1183000,1843000,2759000,3753000,4649000,5937000

    dw 3,7,9,13,28,38,130,220,476,778,1001,1442,2258,2793,4037,5635,8226
    dw 11542,14430,15257,17800
    data ends

    table segment
    db 21 dup ('year summ ne ?? ')
    table ends

    stack segment
    db 16 dup (0)
    stack ends

    codeseg segment
    start:
        mov ax,data
        mov ds,ax

        mov ax,stack
        mov ss,ax
        mov sp,10H

        mov ax,table
        mov es,ax

        mov bx,0
        mov bp,0
        mov di,0A8H
        mov cx,15H
    s:
        push cx

        mov cx,4
        mov si,0
        i:
        mov dl,ds:[bp+si]
        mov es:[bx+si],dl
        inc si
        loop i

        push bp
        add bp,54H
        mov dx,ds:[bp]
        mov es:[bx+5],dx
        mov dx,ds:[bp+2]
        mov es:[bx+7],dx
        pop bp

        mov dx,ds:[di]
        mov es:[bx+0AH],dx
        
        mov DX,es:[bx+7]
        mov AX,es:[bx+5]
        div word ptr es:[bx+0AH]
        mov es:[bx+0DH],AX

        add bx,10H
        add bp,4H
        add di,2H
        pop cx
    loop s

        mov ax,4c00H
        int 21H
    codeseg ends

    end start
    ```

## 实验 8 分析一个奇怪的程序

- 分析下面的程序，在运行前思考，这个程序可以正确返回吗？  
    运行后再思考：为什么是这种结果？  
    通过这个程序加深对相关内容的理解。
    ```x86asm
    assume cs:codeseg

    codeseg segment

        mov ax,4c00h    ;0A77:0000
        int 21h

    start:    
        mov ax,0
    s:  nop             ;0A77:0008
        nop             ;0A77:0009

        mov di,offset s ;0A77:000A ;mov di,0008
        mov si,offset s2           ;mov si,0020
        mov ax,cs:[si]             ;mov ax,F6EB
        mov cs:[di],ax             ;0A77:0008 EB
                                   ;0A77:0009 F6
    s0: jmp short s     ;0A77:0016 EBF0

    s1: mov ax,0        ;0A77:0018
        int 21h
        mov ax,0
                        ;注意这里在编译的时候已经计算好了offset
    s2: jmp short s1    ;0A77:0020 EBF6 ;F6h = -10 
        nop             ;0A77:0022 NOP

    codeseg ends

    end start
    ```
    可以正确返回  
    - 这里根据博主机器上跑的数据做了注释  
    - 程序的入口为 mov ax, 0  
    - 注意到指令jmp short s1占2字节  
    于是指令
        ```
        mov di,offset s
        mov si,offset s2
        mov ax,cs:[si]
        mov cs:[di],ax
        ```
        相当于使用通用寄存器AX做中转将s2处的指令复制到s处  
    - 最后再跳转至s处执行复制过来的指令  
        注意jmp short s1是段内转移的短转移，其只修改IP寄存器    
        而编译的时候已经计算好了offset：补码表示的8bit的有符号数 F6h，即-10  
        通过Debug可知s的段内偏移地址为 8  
        读取s2处复制过来的指令后，IP指向10，
        所以在执行该指令后,IP将指向0  
        因此可以正确返回  

## 实验 9 根据材料编程
- 编程：在屏幕中间分别显示绿色、绿底红色、白底蓝色的字符串'welcome to masm!'。  
    编程所需的只是通过阅读、分析书本上的材料获得。
    - 显示缓冲区B8000h-BFFFFh共32KB的空间
    - 我们将字符写入第0页的显示缓冲区中  
        即B8000h-B8F9Fh共FA0h = 4000个字节
    - 1个字符在显示缓冲区中占用2个字节，低字节存储ASCII码，高字节存储字符的属性  
        可以看出，在显示缓冲区中，偶数地址存放字符，奇数地址存放字符属性
    - 根据字符属性的格式，我们可以给出三个字符串所需要写入的属性：  
        绿色:02h、绿底红色:24h、白底蓝色:71h
    - 编写这个程序遇到一些新手容易忘记的问题：
        - 16进制数字字母开头时，忘记加前导0。  
            报error A2006 undefined symbol  
            例如0B800h错写为B800h
        - 不可以以'idata:[]'方式寻址
        - 在使用BP进行寄存器间接寻址时，缺省的段地址是SS  

    下面是代码：
    ```x86asm
    assume cs:code

    data segment
    db 'welcome to masm!'
    db 02H,24H,71H
    data ends

    stack segment
    db 10H dup (0)
    stack ends

    code segment
    start:
        mov ax,data
        mov ds,ax 
        
        mov ax,0b878H
        mov es,ax

        mov ax,stack
        mov ss,ax
        mov sp,10H

        mov cx,3
        mov bx,10H
        mov bp,0
    row:
        push cx
        mov cx,10H
        mov di,0
        mov si,0
        mov al,[bx]
    column:
        mov dl,[si]
        mov es:[bp+di+40H],dl
        inc di
        mov es:[bp+di+40H],al
        inc si
        inc di
    loop column
        pop cx
        inc bx
        add bp,0a0H
    loop row

        mov ax,4c00H
        int 21H
    code ends

    end start
    ```
    ![LAB9 Result](http://q7qon7hdm.bkt.clouddn.com/images/asmLab/lab9_result.jpg)

## 实验 10 编写子程序

- 这三个子程序将后面的课程设计 1的难度曲线变得更平滑  
    博主刚开始因为子程序编写的鲁棒性不够高，没有进行更多的测试便整合进了课程设计 1，导致后面花费了大量的时间调试，请引以为鉴！
1. 显示字符串
    ```x86asm
    assume cs:code

    data segment
        db 'Welcome to masm!',0
    data ends

    stack segment
        dw 10h dup (0)
    stack ends

    code segment
    start:
        mov ax,stack
        mov ss,ax
        mov sp,10h

        mov ax,data
        mov ds,ax

        mov dh,8
        mov dl,3
        mov cl,2
        mov si,0
        call show_str

        mov ax,4c00h
        int 21h

    show_str:
        push ax
        push bx
        push cx
        push dx
        push di
        push si

        mov ax,0B800H
        mov es,ax

        mov al,2
        mul dl
        mov di,ax
        mov al,0a0h
        mul dh
        mov bx,ax
        add bx,di
        
        mov di,0
        mov al,cl

    str_loop:
        mov cx,0
        mov cl,[si]
        jcxz ok

        mov ch,al
        mov es:[bx+di],cx
        
        inc si
        add di,2

        jmp short str_loop
    ok:
        pop si
        pop di
        pop dx
        pop cx
        pop bx
        pop ax
    ret

    code ends

    end start
    ```
2. 解决除法溢出的问题
    ```x86asm
    assume cs:code 

    stack segment
        dw 10h dup (0)
    stack ends

    code segment

    start:
        mov ax,4240h
        mov dx,000fh
        mov cx,000ah
        call divdw

        mov ax,4c00h
        int 21h
    
    ;因为怕DOS无法显示中文，所以用蹩脚的英文写了注释
    ;dword size div without overflow
    ;parameter:
    ;   Dividend = dx<<16 + ax
    ;   register ax ==> the low  2Bytes of the Dividend
    ;   register dx ==> the high 2Bytes of the Dividend
    ;   register cx ==> Divisor
    ;return:
    ;   Quotient = dx<<16 + ax
    ;   register ax ==> the low  2Bytes of the Quotient
    ;   register dx ==> the high 2Bytes of the Quotient
    ;   register cx ==> Remainder
    divdw:
        push bx

        push ax

        mov ax,dx
        mov dx,0
        div cx

        mov bx,ax
        
        pop ax
        div cx
        mov cx,dx
        mov dx,bx

        pop bx
    ret

    code ends

    end start

    ```
3. 数值显示
    ```x86asm
    assume cs:code

    data segment
        dw 10h dup (0)
    data ends

    stack segment
        dw 10h dup (0)
    stack ends

    code segment

    start:
        mov bx,data
        mov ds,bx
        mov si,0

        mov bx,stack
        mov ss,bx
        mov sp,10h

        mov ax,12666
        call dtoc   

        mov dh,8
        mov dl,3
        mov cl,2
        call show_str

        mov ax,4c00h
        int 21h

    ;因为怕DOS无法显示中文，所以用蹩脚的英文写了注释
    ;digit to char word version
    ;trans word size(2Bytes) digit to C char* style String
    ;parameter:
    ;   register ax ==> digit
    ;   ds:si       ==> the address of the String
    ;return:
    ;   void
    dtoc:
        push bx
        push cx
        push dx
        push di
        push si

        mov bx,10
        mov di,0
    loopDiv:
        mov dx,0
        div bx
        inc di
        add dx,30h
        push dx

        mov cx,ax
        inc cx
    loop loopDiv

        mov cx,di
    fillChar:
        pop dx
        mov ds:[si],dl
        inc si
    loop fillChar

        mov byte ptr ds:[si],0

        pop si
        pop di
        pop dx
        pop cx
        pop bx
    ret

    show_str:
        push ax
        push bx
        push cx
        push dx
        push di
        push si

        mov ax,0B800H
        mov es,ax

        mov al,2
        mul dl
        mov di,ax
        mov al,0a0h
        mul dh
        mov bx,ax
        add bx,di
        
        mov di,0
        mov al,cl

    str_loop:
        mov cx,0
        mov cl,[si]
        jcxz ok

        mov ch,al
        mov es:[bx+di],cx
        
        inc si
        add di,2

        jmp short str_loop
    ok:
        pop si
        pop di
        pop dx
        pop cx
        pop bx
        pop ax
    ret

    code ends

    end start
    ```

## 课程设计 1

- 任务：将[实验 7](#toc-heading-8)中的Power idea公司的数据在屏幕上显示出来。  
    注意，因为程序要显示的数据有些以及大于 2^16-1 = 65535,应该编写一个新的数据到字符串转化的子程序，完成dowrd类型数据到字符串的转化。 
- 这里对[实验 7](#toc-heading-8)的相关代码进行了重构
- 代码中博主采用了两个不同的子程序dtoc、dtocDw来完成不同的数据类型的转换。  
    **实际上不用这么麻烦：**  
    完全可以采用在data段预留足够的空格串的区域，而dtoc只需要填写对应的十进制数字的ASCII即可，不需要主动填充空白字符(fillSpace、fillSpaceDw)。  
    这样只需要编写一个dtoc即可，也不需要额外的参数，实际显示的效果也会更加美观。  
    注：预留足够的数据空间由调用者保证
- 编写这个程序遇到的一些问题：  
    - CX为0时，直接进入循环执行loop指令，CX会溢出至FFFFh  
        dtoc中的jcxz overFill便是为了解决这个问题
    - 子程序设计时注意保存现场，即利用栈保护各个会破坏原有值的寄存器  
        这里遇到了调用show_str无法显示多行的问题
    - 注意栈操作越界的问题，由于8086汇编中确保栈操作合法的任务交给了程序设计者，因此栈操作越界可能引起一系列严重的问题。  
        这里遇到了SP初始值设置错误引起的栈操作越界进而修改code段数据的问题  

```x86asm
assume cs:code

source segment
db '1975','1976','1977','1978','1979','1980','1981','1982','1983'
db '1984','1985','1986','1987','1988','1989','1990','1991','1992'
db '1993','1994','1995'

dd 16,22,382,1356,2390,8000,16000,24486,50065,97479,140417,197514
dd 345980,590827,803530,1183000,1843000,2759000,3753000,4649000,5937000

dw 3,7,9,13,28,38,130,220,476,778,1001,1442,2258,2793,4037,5635,8226
dw 11542,14430,15257,17800
source ends

table segment
    db 21 dup ('year summ ne ?? ')
            ;0123456789012345
table ends

data segment
    db 21 dup ('year 0123456789 01234 01234 ')
            ;0123456789012345678901234567
data ends

stack segment
    dw 20h dup (0)
stack ends

code segment

start:
    mov ax,source
    mov es,ax

    mov ax,table
    mov ds,ax

    mov ax,stack
    mov ss,ax
    mov sp,40h

    call trans

    mov ax,table
    mov es,ax

    mov ax,data
    mov ds,ax

;trans table to C char* style String
    mov cx,21
    mov bx,0
    mov bp,0
toStr:
    ;copy year field from table to data
    ;storage in [bx].0--[bx].3
    ;data source es:[bp].0--es:[bp].3
    mov dx,es:[bp].0
    mov [bx].0,dx
    mov dx,es:[bp].2
    mov [bx].2,dx
    ;insert '|' between year filed and income field
    mov byte ptr [bx].4,124
    ;call dtoc dword version
    ;storage in [bx].5--[bx].14
    ;data source es:[bp].5--es:[bp].8
    add si,5
    mov ax,es:[bp].5
    mov dx,es:[bp].7
    call dtocDw
    ;insert '|' between income field and employee field
    mov byte ptr [bx].15,124
    ;call dtoc word version
    ;storage in [bx].16--[bx].20
    ;data source es:[bp].10--es:[bp].11
    add si,11
    mov ax,es:[bp].10
    call dtoc
    ;insert '|' between employee field and avenger field
    mov byte ptr [bx].21,124
    ;call dtoc word version
    ;storage in [bx].22--[bx].26
    ;data source es:[bp].13--es:[bp].14
    add si,6
    mov ax,es:[bp].13
    call dtoc
    ;insert '\0' at the end
    mov byte ptr [bx].27,0

    add si,6
    add bx,28
    add bp,16
loop toStr

;show data segment
    mov cx,21
    mov si,0
    mov dh,2
    mov dl,26
showData:
    push cx
    mov cl,2
    call show_str
    add si,28
    inc dh
    pop cx
loop showData

    mov ax,4c00H
    int 21H

;trans source to table
;storage in table segent
trans:
    push ax
    push bx
    push cx
    push dx
    push bp
    push si

    mov bp,0

    mov cx,21
    mov bx,0
year:
    push cx

        mov cx,4
        mov si,0
    yLoop:
        mov dl,es:[bp+si]
        mov [bx].0[si],dl
        inc si
    loop yLoop

    add bp,4
    add bx,10h
    pop cx
loop year

    mov cx,21
    mov bx,0
income:
    push cx

        mov cx,2
        mov si,0
    iLoop:
        mov dx,es:[bp+si]
        mov [bx].5[si],dx
        add si,2
    loop iLoop

    add bp,4
    add bx,10h
    pop cx
loop income

    mov cx,21
    mov bx,0
employee:
    push cx
    
    mov si,0
    mov dx,es:[bp+si]
    mov [bx].10[si],dx

    add bp,2
    add bx,10h
    pop cx
loop employee

    mov cx,21
    mov bx,0
calc:
    push cx
    
    mov ax,[bx].5
    mov dx,[bx].7
    mov bp,[bx].10
    div bp
    mov [bx].13,ax

    add bx,10h
    pop cx
loop calc

    pop si
    pop bp
    pop dx
    pop cx
    pop bx
    pop ax
ret

;digit to char word version
;trans word size(2Bytes) digit to C char* style String
;parameter:
;   register ax ==> digit
;   ds:si       ==> the address of the String
;return:
;   void
dtoc:
    push bx
    push cx
    push dx
    push di
    push si

    mov bx,10
    mov di,0
loopDiv:
    mov dx,0
    div bx
    inc di
    add dx,30h
    push dx

    mov cx,ax
    inc cx
loop loopDiv

    mov cx,di
fillChar:
    pop dx
    mov ds:[si],dl
    inc si
loop fillChar

    mov cx,5
    sub cx,di
    jcxz overFill
fillSpace:
    mov byte ptr ds:[si],32
    inc si
loop fillSpace

overFill:
    mov byte ptr ds:[si],0

    pop si
    pop di
    pop dx
    pop cx
    pop bx
ret

;digit to char dword version
;trans dword size(4Bytes) digit to C char* style String
;parameter:
;   register ax ==> the low  2Bytes of the digit
;   register dx ==> the high 2Bytes of the digit
;   ds:si       ==> the address of the String
;return:
;   void
dtocDw:
    push cx
    push dx
    push di
    push si

    mov di,0
loopDivDw:
    mov cx,10
    call divdw
    inc di
    add cx,30h
    push cx

    mov cx,ax
    inc cx
loop loopDivDw

    mov cx,di
fillCharDw:
    pop dx
    mov ds:[si],dl
    inc si
loop fillCharDw

    mov cx,10
    sub cx,di
    jcxz overFillDw
fillSpaceDw:
    mov byte ptr ds:[si],32
    inc si
loop fillSpaceDw

overFillDw:
    mov byte ptr ds:[si],0

    pop si
    pop di
    pop dx
    pop cx
ret

;dword size div without overflow
;parameter:
;   Dividend = dx<<16 + ax
;   register ax ==> the low  2Bytes of the Dividend
;   register dx ==> the high 2Bytes of the Dividend
;   register cx ==> Divisor
;return:
;   Quotient = dx<<16 + ax
;   register ax ==> the low  2Bytes of the Quotient
;   register dx ==> the high 2Bytes of the Quotient
;   register cx ==> Remainder
divdw:
    push bx

    push ax

    mov ax,dx
    mov dx,0
    div cx

    mov bx,ax
    
    pop ax
    div cx
    mov cx,dx
    mov dx,bx

    pop bx
ret

show_str:
    push ax
    push bx
    push cx
    push dx
    push di
    push si

    mov ax,0B800H
    mov es,ax

    mov al,2
    mul dl
    mov di,ax
    mov al,0a0h
    mul dh
    mov bx,ax
    add bx,di
    
    mov di,0
    mov al,cl

str_loop:
    mov cx,0
    mov cl,[si]
    jcxz ok

    mov ch,al
    mov es:[bx+di],cx
    
    inc si
    add di,2

    jmp short str_loop
ok:
    pop si
    pop di
    pop dx
    pop cx
    pop bx
    pop ax
ret

code ends

end start
```
![Course Design 1](http://q7qon7hdm.bkt.clouddn.com/images/asmLab/CourseDesign1.jpg)

## 实验 11 编写子程序

- 未完