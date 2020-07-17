---
title: Assembly Language Lab
top: false
cover: false
img: 'http://bucket.jiabi.tech/medias/featureimages/asm.jpg'
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
    ![LAB1_3 Result](http://bucket.jiabi.tech/images/asmLab/lab1_3.jpg)

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
    ![IMG 3.19](http://bucket.jiabi.tech/images/asmLab/img3.19.jpg)  
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
    ![PSP](http://bucket.jiabi.tech/images/asmLab/lab3_result_u_ds.jpg)

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
    ![LAB6_2 Result](http://bucket.jiabi.tech/images/asmLab/lab6_2_result.jpg)

## 实验 7 寻址方式在结构化数据访问中的应用

- 编程，将 data 段中的数据按如下格式写入到 table 段中，并计算21年中的人均收入(取整)，结果也按照下面的格式保存在table段中。  
    ![LAB7 格式](http://bucket.jiabi.tech/images/asmLab/lab7_format.jpg)  
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
    ![LAB9 Result](http://bucket.jiabi.tech/images/asmLab/lab9_result.jpg)

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
![Course Design 1](http://bucket.jiabi.tech/images/asmLab/CourseDesign1.jpg)

## 实验 11 编写子程序

- 编写一个子程序，将包含任意字符，以 0 结尾的字符串中的小写字母转变成大写字母，描述如下。  
    名称：letterc  
    功能：将以 0 结尾的字符串中的小写字母转变成大写字母  
    参数：ds:si指向字符串首地址  

```x86asm
assume cs:code

data segment
    db "Beginner's All-purpose Symbolic Instrcution Code.",0
data ends

stack segment
    dw 10h dup (0)
stack ends

code segment

begin:
    mov ax,data
    mov ds,ax

    mov ax,stack
    mov ss,ax
    mov sp,10h

    mov si,0
    call letterc

    mov dh,8
    mov dl,3
    mov cl,2
    mov si,0
    call show_str

    mov ax,4c00h
    int 21h

letterc:
    push si
    push cx
    pushf

loopLetter:
    mov ch,0
    mov cl,[si]
    cmp cl,61h
    jb nextLetter
    cmp cl,7ah
    ja nextLetter
    and cl,0dfh
    mov [si],cl

nextLetter: 
    inc si
    inc cx
loop loopLetter

    popf
    pop cx
    pop si
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

end begin
```

## 实验 12 编写0号中断的处理程序

- 编写0号中断的处理程序，使得除法溢出发生时，在屏幕中间显示字符串"divide error!"，然后返回到DOS。
- 这里没有对中断处理程序使用到的寄存器进行保护

```x86asm
assume cs:code

code segment

start:
    mov ax,cs
    mov ds,ax
    mov si,do0

    mov ax,0
    mov es,ax
    mov di,200h

    mov cx,do0end-do0
    cld
    rep movsb

    mov word ptr es:[4*0],200h
    mov word ptr es:[4*0+2],0

    mov ax,4c00h
    int 21h

do0:
    jmp do0start
    db "divide error!"
do0start:
    mov ax,cs
    mov ds,ax
    mov si,202h

    mov ax,0b800h
    mov es,ax
    mov di,12*160+34*2

    mov cx,13
    S:
    mov al,[si]
    mov es:[di],al
    inc di
    mov byte ptr es:[di],2
    inc di
    inc si    
    loop s

    mov ax,4c00h
    int 21h
do0end:
    nop

code ends

end start
```

## 实验 13 编写、应用中断例程

1. 编写并安装int 7ch中断例程，功能为显示一个用0结束的字符串，中断例程安装在0:200处。  
    参数：(dh)=行号 (dl)=列号 (cl)=颜色 ds:si指向字符串首地址

- 中断例程

```x86asm
assume cs:code

code segment

start:
    mov ax,cs
    mov ds,ax
    mov si,do7c

    mov ax,0
    mov es,ax
    mov di,200h

    mov cx,do7cend-do7c
    cld
    rep movsb

    mov word ptr es:[4*7ch],200h
    mov word ptr es:[4*7ch+2],0

    mov ax,4c00h
    int 21h 

do7c:
    push ax
    push cx
    push si
    push di
    push es    

    mov ax,0b800h
    mov es,ax
    mov di,0
    mov al,160
    mul dh
    add di,ax
    mov al,2
    mul dl
    add di,ax

    mov al,cl
s: 
    mov cx,0
    mov cl,ds:[si]
    jcxz ok
    mov ch,al
    mov es:[di],cx

    inc si
    add di,2

    jmp s
ok:
    pop es
    pop di
    pop si
    pop cx
    pop ax

    iret
do7cend:
    nop

code ends
end start
```

- 测试代码

```x86asm
;TEST LAB13_1
assume cs:code

data segment
    db "Welcome to masm!",0
data ends

stack segment
    dw 10h dup (0)
stack ends

code segment

start:
    mov ax,data
    mov ds,ax

    mov ax,stack
    mov ss,ax
    mov sp,10h

    mov dh,10
    mov dl,10
    mov cl,2
    mov si,0
    int 7ch
    
    mov ax,4c00h
    int 21h

code ends
end start
```

2. 编写并安装int 7ch中断例程，功能为完成loop指令的功能。  
    参数：(cx)=循环次数 (bx)=位移

- 中断例程

```x86asm
assume cs:code

code segment

start:
    mov ax,cs
    mov ds,ax
    mov si,do7c

    mov ax,0
    mov es,ax
    mov di,200h

    mov cx,do7cend-do7c
    cld
    rep movsb

    mov word ptr es:[4*7ch],200h
    mov word ptr es:[4*7ch+2],0

    mov ax,4c00h
    int 21h 

do7c:
    dec cx
    jcxz ok
    push bp
    mov bp,sp
    add [bp+2],bx
    pop bp
ok:
    iret
do7cend:
    nop

code ends
end start
```

- 测试代码

```x86asm
;TEST LAB13_2
assume cs:code

stack segment
    dw 10h dup (0)
stack ends

code segment

start:
    mov ax,stack
    mov ss,ax
    mov sp,10h

    mov ax,0b800h
    mov es,ax
    mov di,160*12
    mov bx,s-sEnd
    mov cx,80
s:
    mov byte ptr es:[di],'!'
    inc di
    mov byte ptr es:[di],4
    inc di
    int 7ch
sEnd:
    nop

    mov ax,4c00h
    int 21h

code ends
end start
```

3. 下面的程序，分别在屏幕的第2、4、6、8行显示4句英文诗，补全程序。

```x86asm
assume cs:code

code segment

    s1: db 'Good,better,best,','$'
    s2: db 'Never let it rest,','$'
    s3: db 'Till good is better,','$'
    s4: db 'And better,best.','$'
    s : dw offset s1,offset s2,offset s3,offset s4
    row: db 2,4,6,8
start:
    mov ax,cs
    mov ds,ax
    mov bx,offset s
    mov si,offset row
    mov cx,4
ok:
    mov bh,0
    mov dh,ds:[si]
    mov dl,0
    mov ah,2
    int 10h

    mov dx,ds:[bx]
    mov ah,9
    int 21h
    inc si
    add bx,2
    loop ok

    mov ax,4c00h
    int 21h

code ends

end start
```

## 实验 14 访问CMOS RAM

- 编程，以"年/月/日 时:分:秒"的格式，显示当前的日期、时间。
    - 不得不说，这本书的曲线是相当顺滑的  
    在实验13.3中，王爽老师把实验14可能要用到的技巧以实验的形式呈现
    - 博主出了个挺搞笑的Bug，因为忘记加上"inc si"  
    导致显示的时间是"20/20/20 20:20:20",还觉得好巧 :joy:
    - 查验结果是否正确可以用date与time命令

    ```x86asm
    assume cs:code

    stack segment
        dw 10h dup (0)
    stack ends

    code segment
        
        dataPos: db 9,8,7,4,2,0
    start:
        mov ax,stack
        mov ss,ax
        mov sp,10h

        mov ax,cs
        mov ds,ax
        mov si,offset dataPos

        mov bx,0b800h
        mov es,bx
        mov di,160*12+32*2

        mov cx,6
    nextField:
        push cx
        
        mov al,ds:[si]
        inc si
        out 70h,al
        in al,71h

        mov ah,al
        mov cl,4
        shr ah,cl
        and al,0fh
        
        add ah,30h
        add al,30h

        mov es:[di],ah
        inc di
        mov byte ptr es:[di],2
        inc di
        mov es:[di],al
        inc di
        mov byte ptr es:[di],2
        inc di

        pop cx
        
        cmp cx,1
        je doNotShow
        
        cmp cx,4
        mov ah,32   ;ASCII 32 (space)
        je show
        mov ah,47   ;ASCII 47 /
        ja show
        mov ah,58   ;ASCII 58 :
        jb show
    show:
        mov es:[di],ah
        inc di
        mov byte ptr es:[di],2
        inc di

    doNotShow:
        loop nextField

        mov ax,4c00h
        int 21h

    code ends

    end start
    ```

## 实验 15 安装新的int9中断例程

- 安装一个新的int 9中断例程  
    功能：在DOS下，按下'A'键后，除非不再松开，如果松开，就显示满屏幕的'A'，其他键照常处理。
- 这里没有写恢复中断向量的程序，感觉用了这个会被逼疯掉 233

```x86asm
assume cs:code

stack segment
    dw 20h dup (0)
stack ends

code segment

start:
    mov ax,stack
    mov ss,ax
    mov sp,20h

    push cs
    pop ds

    mov ax,0
    mov es,ax
    mov di,200h

    push es:[4*9]
    pop es:[di]
    add di,2
    push es:[4*9+2]
    pop es:[di]
    add di,2

    mov si,newInt9
    mov cx,newInt9End-newInt9
    cld
    rep movsb

    cli
    mov word ptr es:[4*9],204h
    mov word ptr es:[4*9+2],0
    sti

    mov ax,4c00h
    int 21h

newInt9:
    push ax
    push bx
    push cx
    push es

    in al,60h

    pushf
    call dword ptr cs:[200h]

    cmp al,10011110b
    jne notA

    mov ax,0b800h
    mov es,ax
    mov bx,0
    mov cx,2000
    s:
        mov byte ptr es:[bx],41h
        inc bx
        mov byte ptr es:[bx],2
        inc bx
    loop s

notA:
    pop es
    pop cx
    pop bx
    pop ax
    iret
newInt9End:
    nop;

code ends

end start
```

## 实验 16 编写包含多个功能子程序的中断例程

- 安装一个新的int 7ch中断例程，为显示输出提供如下功能子程序：
    1. 清屏
    2. 设置前景色
    3. 设置背景色
    4. 向上滚动一行
- 入口参数说明如下：
    1. 用AH寄存器传递功能号：  
        0 清屏
        1 设置前景色
        2 设置背景色
        3 向上滚动一行  
    2. 对于1、2号功能，用AL传送颜色值，AL∈{0,1,2,3,4,5,6,7}
- 这里需要注意偏移地址的问题:  
    - 将中断例程放在最前面  
    - 入口地址改为20:0而不是0:200
    - 这里也可以使用org伪指令，更简便

```x86asm
assume cs:code

stack segment
    dw 20h dup (0)
stack ends

code segment

int7c:
    jmp short int7cBegin

    table dw cls,setFgColor,setBgColor,scrollUp

    int7cBegin:
        push bx

        cmp ah,3
        ja int7cRet
        
        mov bl,ah
        mov bh,0
        add bx,bx
        call word ptr table[bx]

    int7cRet:
        pop bx
    iret

    cls:
        push bx
        push cx
        push es

        mov bx,0b800h
        mov es,bx
        mov bx,0
        mov cx,2000
    clsLoop:
        mov byte ptr es:[bx],' '
        add bx,2
        loop clsLoop

        pop es
        pop cx
        pop bx
    ret

    setFgColor:
        push bx
        push cx
        push es

        mov bx,0b800h
        mov es,bx
        mov bx,1
        mov cx,2000
    setFgLoop:
        and byte ptr es:[bx],11111000b
        or byte ptr es:[bx],al
        add bx,2
        loop setFgLoop

        pop es
        pop cx
        pop bx
    ret
    
    setBgColor:
        push bx
        push cx
        push es
        
        mov cl,4
        shl al,cl

        mov bx,0b800h
        mov es,bx
        mov bx,1
        mov cx,2000
    setBgLoop:
        and byte ptr es:[bx],10001111b
        or byte ptr es:[bx],al
        add bx,2
        loop setBgLoop

        pop es
        pop cx
        pop bx
    ret
    
    scrollUp:
        push si
        push di
        push ds
        push es
        push ax
        push cx

        mov ax,0b800h
        mov ds,ax
        mov si,160
        mov es,ax
        mov di,0

        cld
        mov cx,24
    scrollUpLoop:
        push cx
        mov cx,160
        rep movsb
        pop cx
        loop scrollUpLoop

        mov cx,80
    clear:
        mov byte ptr es:[di],' '
        add di,2
        loop clear

        pop cx
        pop ax
        pop es
        pop ds
        pop di
        pop si
    ret

int7cEnd:
    nop

start:
    mov ax,stack
    mov ss,ax
    mov sp,20h

    push cs
    pop ds
    mov si,int7c
    mov ax,20h
    mov es,ax
    mov di,0

    mov cx,int7cEnd-int7c
    cld
    rep movsb

    ;install int 7ch
    mov ax,0
    mov es,ax
    mov word ptr es:[4*7ch],0
    mov word ptr es:[4*7ch+2],20h

    ;Test int 7ch
    mov ah,0
    int 7ch

    mov ah,1
    mov al,2
    int 7ch

    mov ah,2
    mov al,4
    int 7ch

    mov ah,3
    int 7ch
    
    mov ax,4c00h
    int 21h
    
code ends

end start
```

## 实验 17 编写包含多个功能子程序的中断例程

- 安装一个新的int 7ch中断例程，实现通过逻辑扇区号对软盘进行读写。  
    参数说明：
    1. 用AH寄存器传递功能号：0表示读 1表示写；
    2. 用DX寄存器传递要读写的扇区的逻辑扇区号(0-2879)；
    3. 用ES:BX指向存储读出数据或写入数据的内存区。
- 遇到的Bug：  
    因为是直接复制用的[实验 16](#toc-heading-18)的框架  
    其中第15行开始，修改了BX的值，而BX又是int 7ch的参数，但是没有在调用子程序之前恢复其值  
    导致向软盘写入数据的时候，偏移地址不对，会少写两个字节的数据。  
    还是要注意保护寄存器的值啊！

```x86asm
assume cs:code

data segment
    db 'Test int7ch:read from floppy sectors.','$'
data ends

backup segment
    db 512 dup (0)
backup ends

stack segment
    dw 20h dup (0)
stack ends

code segment

start:
    mov ax,stack
    mov ss,ax
    mov sp,20h

    push cs
    pop ds
    mov si,int7c
    mov ax,0
    mov es,ax
    mov di,200h

    mov cx,int7cEnd-int7c
    cld
    rep movsb

    ;install int 7ch
    mov ax,0
    mov es,ax
    mov word ptr es:[4*7ch],200h
    mov word ptr es:[4*7ch+2],0

    ;test int 7ch

    ;backup sector 1024
    mov ax,backup
    mov es,ax

    mov ah,0
    mov dx,1024
    mov bx,0
    int 7ch
    
    push es
    ;test begin
    mov ax,data
    mov es,ax
    mov ds,ax

    mov ah,2
    mov bh,0
    mov dh,8
    mov dl,30
    int 10h
    mov ah,9
    mov dx,0
    int 21h

    mov ah,1
    mov dx,1024
    mov bx,0
    int 7ch

    ;change data
    mov byte ptr es:[0],'!'

    mov ah,0
    mov dx,1024
    mov bx,0
    int 7ch

    mov ah,2
    mov bh,0
    mov dh,12
    mov dl,30
    int 10h
    mov ah,9
    mov dx,0
    int 21h

    ;test end
    pop es

    ;recovery sector 1024
    mov ah,1
    mov dx,1024
    mov bx,0
    int 7ch

    mov ax,4c00h
    int 21h

    org 200h
int7c:
    jmp short int7cBegin

    table dw rSector,wSector

    int7cBegin:
        push si
        
        cmp ah,1
        ja int7cRet

        push bx
        mov bl,ah
        mov bh,0
        add bx,bx
        mov si,bx
        pop bx
        call word ptr table[si]

    int7cRet:
        pop si
    iret

    rSector:
        push ax
        push cx
        push dx

        call getPhy
        mov ah,2
        mov al,1
        int 13h

        pop dx
        pop cx
        pop ax
    ret

    wSector:
        push ax
        push cx
        push dx

        call getPhy
        mov ah,3
        mov al,1
        int 13h

        pop dx
        pop cx
        pop ax
    ret

    getPhy:
        mov ax,dx
        mov dx,0
        mov cx,1440
        div cx
        push ax ;storage int(dx/1440)
        
        mov ax,dx
        mov cl,18
        div cl

        mov ch,al
        mov cl,ah
        inc cl

        pop dx
        mov dh,dl
        mov dl,0
    ret

int7cEnd:
    nop

code ends

end start
```

## 课程设计 2

- 要求不列了,最后还是烂尾了😭
- 完成情况
    - 功能1、2
    - 功能3的ESC退出功能点没有完成  
        按ESC会造成严重错误从而直接退出虚拟机  
        找了很久的Bug都没什么头绪
    - 功能4没有对输入进行必要的验证  
        交互不够友好
- 虽然大部分都是之前已经写好的程序，但还是遇到了很多问题。其中值得记录的有
    - 搭建程序框架时，死活读不到程序  
    原因是对伪指令org 7c00h的误解  
    编译器实际上会把代码放到cs段的7c00h处，而不是仅仅计算时候以7c00h为偏移  
    把org 7c00h放到boot之后，导致8行计算mov bx，boot时候，实际是一个很小的值
    - 编写功能2时，死活没法重启，直接跑崩  
    一度怀疑是电脑出了问题，经过很久的debug，终于找到了原因  
    SS设置在了7c00h之后，而系统引导程序依赖于bios设置的stack  
    这样会导致栈区和代码重合，进而修改系统引导程序的代码

```x86asm
assume cs:code

code segment

start:
    mov ax,cs
    mov es,ax
    mov bx,boot

    mov ah,3
    mov al,1
    mov ch,0
    mov cl,1
    mov dh,0
    mov dl,0
    int 13h

    mov ax,cs
    mov es,ax
    mov bx,bootLoader

    mov ah,3
    mov al,3
    mov ch,0
    mov cl,2
    mov dh,0
    mov dl,0
    int 13h

    mov ax,4c00h
    int 21h

    org 7c00h
boot:
    mov ax,0
    mov ss,ax
    mov sp,7c00h

    mov ax,0
    mov ds,ax

    mov ax,0
    mov es,ax
    mov bx,7e00h

    mov ah,2
    mov al,3
    mov ch,0
    mov cl,2
    mov dh,0
    mov dl,0
    int 13h
    
    mov bx,0
	push bx
	mov bx,7e00h 
	push bx
	retf

    org 7e00h
bootLoader:
    jmp bootStart

    option1 db '1) reset pc    ',0
    option2 db '2) start system',0
    option3 db '3) clock       ',0
    option4 db '4) set clock   ',0
    buffer  db '                '
    tip     db 'Press F1 to change color.ESC return menu',0
    menu dw resetPC,startSys,clock,setClock
    dataPos: db 9,8,7,4,2,0
    top dw 0
;###########################################################################
bootStart:
    call showMenu
    call bootFun

    mov ax,4c00h
    int 21h
;###########################################################################
showMenu:
    push ax
    push bx
    push cx
    push dx

    mov dl,2
    call cls

    mov ax,0    
    mov dh,10
    mov dl,32
    mov cl,2
    mov si,offset option1
menuDetail:
    call show_str
    add si,10h
    inc dh
    inc ax
    cmp ax,4
jne menuDetail

    mov ah,2
    mov bh,0
    mov dh,14
    mov dl,32
    int 10h

    pop dx
    pop cx
    pop bx
    pop ax
ret
;###########################################################################
bootFun:
    call clearBuff

    mov dh,14
    mov dl,32 
    mov si,offset buffer
    call getStr

    mov si,offset buffer
    call transDigit
    mov top,0
    cmp dx,0
    jne bootStart   ;dx!=0 input illegal


    dec al          ;else  legall
    cmp al,4
    ja bootFun

    mov bl,al
    mov bh,0
    add bx,bx
    jmp word ptr menu[bx]
;---------------------------------------------------------------------------
resetPC:
    mov bx,0ffffh
    push bx
    mov bx,0
    push bx
    retf
;---------------------------------------------------------------------------
startSys:
    mov ah,2
    mov bh,0
    mov dh,0
    mov dl,0
    int 10h

    mov dl,7
    call cls

    mov ax,0
    mov es,ax
    mov bx,7c00h

    mov ah,2
    mov al,1
    mov ch,0
    mov cl,1
    mov dh,0
    mov dl,80h
    int 13h

    mov bx,0
    push bx
    mov bx,7c00h
    push bx
    retf
;---------------------------------------------------------------------------
clock:
    push ax
    push bx
    push cx
    push dx
    push si

    call instInt9
    mov dl,2
    call cls

    mov ax,0    
    mov dh,13
    mov dl,20
    mov cl,2
    mov si,offset tip
    call show_str

    mov ah,2
    mov bh,0
    mov dh,14
    mov dl,32
    int 10h

showTimeLoop:
    call showTime
jmp showTimeLoop

clockRet:
    call recInt9
    jmp bootStart
;---------------------------------------------------------------------------
setClock:
    push cx
    mov dl,2
    call cls

    call showTime
    
    mov cx,0
setClockLoop:
    call setField
    inc cx
    cmp cx,6
jne setClockLoop
    
    pop cx
    jmp bootStart
    
ret
setField:
    push ax
    push bx
    push cx
    push dx

    mov ah,2
    mov bh,0
    mov dh,12
    mov dl,32
    add dl,cl
    add dl,cl
    add dl,cl
    int 10h

    mov si,offset buffer
    call getStr

    mov si,offset buffer
    mov ah,ds:[si]
    sub ah,30h
    inc si
    mov al,ds:[si]
    sub al,30h
    mov top,0

    and ah,00001111b
    push cx
    mov cl,4
    shl ah,cl
    pop cx
    and al,00001111b
    or al,ah

    push ax
    mov si,offset dataPos
    add si,cx
    mov al,ds:[si]
    out 70h,al
    pop ax
    out 71h,al

setFieldRet:
    pop dx
    pop cx
    pop bx
    pop ax
ret
;###########################################################################
getStr:
    push ax

    call clearBuff
getStrLoop:
    mov ax,top
    cmp ax,2
    je enterKey
    mov ah,0
    int 16h
    cmp al,20h
    jb notChar

    mov ah,0
    call charFun
    mov ah,2
    call charFun

    jmp getStrLoop
notChar:
    cmp ah,0eh
    je backKey
    cmp ah,1ch
    je enterKey
    jmp getStrLoop
backKey:
    mov ah,1
    call charFun
    mov ah,2
    call charFun
    jmp getStrLoop
enterKey:
    mov al,0
    mov ah,0
    call charFun
    mov ah,2
    call charFun

    pop ax
ret

;ah 0 pushChar 1 popChar 2 showChar
charFun:
    jmp short charInit

    table dw pushChar,popChar,showChar

charInit:
    push bx
    push dx
    push di
    push es

    cmp ah,2
    ja charFunRet

    mov bl,ah
    mov bh,0
    add bx,bx
    jmp word ptr table[bx]
    
pushChar:
    mov bx,top
    mov [si][bx],al
    inc top
    jmp charFunRet

popChar:
    cmp top,0
    je charFunRet
    dec top
    mov bx,top
    mov al,[si][bx]
    jmp charFunRet

showChar:
    mov bx,0b800h
    mov es,bx
    mov al,160
    mov ah,0
    mul dh
    mov di,ax
    add dl,dl
    mov dh,0
    add di,dx

    mov bx,0
showCharLoop:
    cmp bx,top
    je isEmpty
    mov al,[si][bx]
    mov es:[di],al
    mov byte ptr es:[di+2],' '
    inc bx
    add di,2
    jmp showCharLoop
    
isEmpty:
    mov byte ptr es:[di],' '
charFunRet:
    pop es
    pop di
    pop dx
    pop bx
ret
;###########################################################################
;dx=0 success dx=1 fail 
transDigit:
    push bx
    push cx
    pushf

    mov ax,0
    mov cx,10
    mov dx,0
    push ax
    popf
transLoop:
    mov bl,byte ptr ds:[si]
    cmp bl,0
    je transRet
    cmp bl,39h
    ja transFail
    cmp bl,30h
    jb transFail
    sub bl,30h
    mul cx
    jb transFail    ;cf=1
    mov bh,0
    add ax,bx
    jb transFail    ;cf=1
    inc si
jmp transLoop

transFail:
    mov dx,1
transRet:
    popf
    pop cx
    pop bx
ret
;###########################################################################
show_str:
    push ax
    push bx
    push cx
    push dx
    push di
    push si
    push es

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
    pop es
    pop si
    pop di
    pop dx
    pop cx
    pop bx
    pop ax
ret
;###########################################################################
;parameter dl->foreground color
cls:
    push bx
    push cx
    push es

    mov bx,0b800h
    mov es,bx
    mov bx,0
    mov cx,2000
clsLoop:
    mov byte ptr es:[bx],' '
    inc bx
    mov byte ptr es:[bx],dl
    inc bx
    loop clsLoop

    pop es
    pop cx
    pop bx
ret
;###########################################################################
clearBuff:
    mov ah,1
	int 16h
	jz clearBuffRet
	mov ah,0
	int 16h
	jmp clearBuff
clearBuffRet:
ret
;###########################################################################
showTime:
    push si
    push di
    push bx
    push es
    push cx
    push ax

    mov si,offset dataPos

    mov bx,0b800h
    mov es,bx
    mov di,160*12+32*2

    mov cx,6
    nextField:
    push cx

    mov al,ds:[si]
    inc si
    out 70h,al
    in al,71h

    mov ah,al
    mov cl,4
    shr ah,cl
    and al,0fh

    add ah,30h
    add al,30h

    mov es:[di],ah
    add di,2
    mov es:[di],al
    add di,2

    pop cx

    cmp cx,1
    je doNotShow

    cmp cx,4
    mov ah,32   ;ASCII 32 (space)
    je show
    mov ah,47   ;ASCII 47 /
    ja show
    mov ah,58   ;ASCII 58 :
    jb show
    show:
    mov es:[di],ah
    add di,2

    doNotShow:
    loop nextField
    
    pop ax
    pop cx
    pop es
    pop bx
    pop di
    pop si
ret
;###########################################################################
instInt9:
    push ds
    push ax
    push bx
    push cx
    push dx
    push di

    push cs
    pop ds

    mov ax,0
    mov es,ax
    mov di,200h

    push es:[4*9]
    pop es:[di]
    add di,2
    push es:[4*9+2]
    pop es:[di]
    add di,2

    mov si,newInt9
    mov cx,newInt9End-newInt9
    cld
    rep movsb

    cli
    mov word ptr es:[4*9],204h
    mov word ptr es:[4*9+2],0
    sti

    pop di
    pop dx
    pop cx
    pop bx
    pop ax
    pop ds
ret

recInt9:
    push ax
    push es

    mov ax,0
    mov es,ax

    cli
    push es:[200h]
    pop es:[4*9]
    push es:[202h]
    pop es:[4*9+2]
    sti

    pop es
    pop ax
ret

newInt9:
    push ax
    push bx
    push cx
    push es

    in al,60h

    pushf
    call dword ptr cs:[200h]

    cmp al,10000001b    ;ESC
    je escFun
    cmp al,10111011b    ;F1
    je f1Fun

    jmp int9Ret

escFun:
    pop es
    pop cx
    pop bx
    pop ax
    add sp,4
    popf
    jmp clockRet

f1Fun:
    mov ax,0b800h
    mov es,ax
    mov bx,1
    mov cx,2000
    f1FunLoop:
        inc byte ptr es:[bx]
        add bx,2
    loop f1FunLoop

int9Ret:
    pop es
    pop cx
    pop bx
    pop ax
    iret
newInt9End:
    nop
;###########################################################################
code ends

end start
```

## 尾声

- 心心念念的"汇编语言 王爽"终于快读完了，总的来说难度不高，读起来也挺有趣
- 体验了当年的DOS（这玩意年龄比我还大 :joy:
    > MS-DOS 6.22 1994年6月，最后一个销售版本
- 体会到软件工程化的必要性以及单元测试、自动化测试的重要性
- 最后还是有缺憾，课程设计2没有全部完成（以后应该会重写
- 另开一篇综合研究
