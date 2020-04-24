---
title: 微机原理与接口技术 课程实验
top: false
cover: false
img: http://q9apk9x38.bkt.clouddn.com/medias/featureimages/code.jpg
toc: true
mathjax: true
date: 2020-03-25 11:06:44
password:
summary: 记录该课程实验相关内容
tags:
    - ASM
    - LAB
    - 课程实验
categories:
    - 计算机基础
---

#  微机原理与接口技术 课程实验

> 微机原理与接口技术  
> The Principle of Microcomputer & Interface Technology
```bash
C:\>ver 
MS-DOS Version 6.22  
C:\MASM>masm
Microsoft (R) MASM Compatibility Driver Version 6.11  
Copyright (C) Microsoft Corp 1993.  A1l rights reserved.
```

## 实验一  Debug 的使用

### 一、实验内容及要求

1. 查看寄存器中的内容。
2. 查看内存中的内容  
    PC 主板中有一个生产日期，在内存 FFF00~FFFFFH 的某几个单元中。请找到这个生产日
    期并试图改变它。
3. 向内存 B8100H 开始的单元中填写数据   
    如：1,2,3,4,5,6,7,8 等等。
4. 使用 DEBUG，将下列的程序段写入内存，逐条执行，观察每条指令执行后 CPU 中相
    关寄存器中的内容变化
    ```x86asm
    mov ax,4e20h
    add ax,1416h
    mov bx,2000h
    add ax,bx
    mov bx,ax
    add ax,bx
    mov ax,001ah
    mov bx,0026h
    add al,bl
    add ah,bl
    add bh,al
    mov ah,0
    add al,bl
    add al,9ch
    ```

###  二、实验原理及步骤

1. 进入 Debug 的方法
2. 用 R 命令查看、改变 CPU 寄存器的内容  
    R 查看寄存器内容  
    改变寄存器的值：R 相关寄存器
3. 用 D 命令查看内存中的内容  
    查看内存中的内容方法： D 段地址：偏移地址  
    用 D 命令查看内存范围的方法： D 段地址：起始偏移地址 结尾偏移地址
4. 改写内存中的内容
    - 方法 1：用 E 命令改写内存中的内容。比如将 2000:0~2000:9 单元中的内容分别写为  
    0、1、2、3、4、5、6、7、8、9。  
    用命令：E 起始地址 数据 数据。。。。的格式。  
    即：e 2000:0 0 1 2 3 4 5 6 7 8 9。
    - 方法 2：也可以逐个逐个的修改： E 2000:0 按回车。
5. 查看内存中原有的机器码所对应的汇编指令  
    方法：U SA:EA
6. 用 T 命令单步执行  
7. 用 A 命令以汇编指令的形式在内存中写入机器指令。
    - 方法一：A SA:EA
    - 方法二：A

###  三、实验结果分析及实验报告

1. LAB1_1: 略
2. LAB1_2:  
    ![LAB1_2 Result](http://q9apk9x38.bkt.clouddn.com/images/TPoM-IT-LAB/lab1_2.jpg)
3. LAB1_3: 显存区域，无现象
4. LAB1_4:  
    ![LAB1_4 Code](http://q9apk9x38.bkt.clouddn.com/images/TPoM-IT-LAB/lab1_4.jpg)
    ![LAB1_4 Result](http://q9apk9x38.bkt.clouddn.com/images/TPoM-IT-LAB/lab1_4_result.jpg)

## 实验二 数据处理实验

### 一、实验内容及要求

1. 编程从键盘上输入一个字符并显示。
2. 编程实现 2 的 4 次方。

###  二、实验原理及步骤

1. 单字符显示
    ```x86asm
    assume cs:code

    code segment
        mov ah,1
        int 21h
        mov dl,al
        mov ah,2
        int 21h
        mov ah,4ch
        int 21h
    code ends

    end
    ```
2. 计算2的4次方
    ```x86asm
    assume cs:code

    code segment
    start: 
        mov ax,2
        add ax,ax
        mov ax,ax
        mov ax,ax
        mov ah,4ch
        int 21h
    code ends

    end start
    ```

###  三、实验结果分析及实验报告

1. LAB2_1:  
    ![LAB2_1 Result](http://q9apk9x38.bkt.clouddn.com/images/TPoM-IT-LAB/lab2_1.jpg)
2. LAB2_2:  
    ![LAB2_2 Result](http://q9apk9x38.bkt.clouddn.com/images/TPoM-IT-LAB/lab2_2.jpg)

## 实验三 分支程序设计

### 一、实验内容及要求

- 某班 20 人，统计成绩 90 分以上，80-89，70-79，60-69 分及 60 分以下的人数
- 当前数据段中 DATA1 开始的 20 个字节单元中，存放某课程的考试成绩
- 统计人数存放在同一数据段的 DATA2 开始的 5 个字节单元中
- 在屏幕上显示统计结果

###  二、实验原理及步骤

```x86asm
assume cs:code, ds:data

data segment
    data1 db 50,61,62,78,65,89,90,76,88,91,52,68,78,95,81,82,75,82,55,87
    data2 db 5 dup (0)
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

    mov cx,20
    lea si, data1
    lea di, data2
again: 
        mov al,[si]

        cmp al,90
        jc next1
        inc byte ptr[di]
        jmp newRound
    next1: 
        cmp al,80
        jc next2
        inc byte ptr[di+1]
        jmp newRound
    next2: 
        cmp al,70
        jc next3
        inc byte ptr[di+2]
        jmp newRound
    next3: 
        cmp al,60
        jc next4
        inc byte ptr[di+3]
        jmp newRound
    next4: 
        inc byte ptr[di+4]

    newRound: 
        inc si
    
loop again

    mov cx,5
    lea di, data2
show_result:     
    call show_str
    inc di
loop show_result

mov ax,4c00h
int 21h

show_str:
    push dx
    push ax
    push cx
    mov cl,byte ptr[di]

    or byte ptr[di],30h
    mov dl,[di]
    mov ah,2
    int 21h

    mov byte ptr[di],cl
    pop cx
    pop ax
    pop dx
ret

code ends

end start
```

###  三、实验结果分析及实验报告

![LAB3 Result](http://q9apk9x38.bkt.clouddn.com/images/TPoM-IT-LAB/lab3_result.jpg)

## 实验四 循环程序设计

### 一、实验内容及要求

- 利用循环编写 10 个数的排序程序，可以选择不同的排序算法实现。
- 这里使用的希尔排序
    - 增量序列使用的是最初Donald Shell给出的建议
    - 这里假设需要排序的数为字类型，需要排序的数量少于2*(2^8-1)

###  二、实验原理及步骤

```x86asm
assume cs:code

data segment
    buf dw 3,1,2,4,5,7,-6,8,-8,10
    num = ($-buf)/2
data ends

stack segment
    dw 20h dup (0)
stack ends

code segment

start:
    mov ax,data
    mov ds,ax

    mov ax,stack
    mov ss,ax
    mov sp,20h

    call shellSort

    mov ax,4c00h
    int 21h

shellSort:
    push ax
    push bx
    push cx
    push dx
    push bp
    push si
    
    mov ax,num
    add ax,ax
    mov si,ax
;next Increment
nextInc:
    mov dl,2
    div dl
    mov ah,0
    div dl
    mov ah,0
    add ax,ax

        mov bx,ax   ;bx = increment
    nextNum:
        mov dx,[bx]

        mov bp,bx   
        cmpNum:
            push bp
            sub bp,ax
            mov cx,ds:[bp]
            pop bp
            cmp dx,cx
            jge getNextNum  ;dx >= cx
            mov ds:[bp],cx
        sub bp,ax
        cmp bp,ax
        jnb cmpNum  ;bp >= ax

        getNextNum:
        mov ds:[bp],dx
    add bx,2
    cmp bx,si
    jb nextNum  ;bx < num

cmp ax,2
jne nextInc ;ax != 2

    pop si
    pop bp
    pop dx
    pop cx
    pop bx
    pop ax
ret

code ends

end start
```

###  三、实验结果分析及实验报告

![LAB4 Result](http://q9apk9x38.bkt.clouddn.com/images/TPoM-IT-LAB/MPLAB4.JPG)