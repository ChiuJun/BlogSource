---
title: AssemblyLanguage Research
top: false
cover: false
img: 'http://q9apk9x38.bkt.clouddn.com/medias/featureimages/asm.jpg'
toc: true
mathjax: true
date: 2020-05-01 16:32:55
password:
summary: 记录王爽汇编语言第三版综合研究相关
tags:
    - ASM
    - LAB
categories:
    - 计算机基础
---

# 王爽《汇编语言》第三版 综合研究

> 这部分内容主要是启发学习者进行独立研究和深度思考

## 环境

```bash
C:\>ver 
MS-DOS Version 6.22  
C:\MASM>masm
Microsoft (R) MASM Compatibility Driver Version 6.11  
Copyright (C) Microsoft Corp 1993.  A1l rights reserved.
C:\TC2>tcc
Turbo C Version 2.0 Copyright (C) 1987, 1988 Borland International
```

## 研究试验 1 搭建一个精简的C语言开发环境

- 为了研究的过程清晰明了，我们的原则是：
    - 我们只运行解决当前问题所要用的，我们已知的程序；
    - 所有我们已知的程序在解决我们的问题的运行过程中，需要用到的程序和文件，也都是我们已知的
- 说人话就是，尽量精简环境，只用一部分必要的模块
- 这里使用TC.EXE以及当前必要的依赖搭建这个精简的环境
    - C0S.OBJ
    - CS.LIB
    - EMU.LIB
    - GRAPHICS.LIB
    - MATHS.LIB

## 研究试验 2 使用寄存器

1. 编一个程序ur1.c

    ```c
    main(){
        _AX=1;
        _BX=1;
        _CX=2;
        _AX=_BX+_CX;
        _AH=_BL+_CL;
        _AL=_BH+_CH;
    }
    ```

2. 用Debug加载ur1.exe，用u命令查看ur1.exe编译后的机器码和汇编代码。  
    思考：main函数的代码在什么段中？用Debug怎样找到ur1.exe中main函数的代码？
    - 可以使用TCC的-S选项生成ASM文件
    - 推测是在CS段中
    - 结合3的结果使用Debug追踪发现，CS:011A处会对main函数所对应的偏移地址调用

3. 用下面的方法打印出ur1.exe被加载运行时，main函数在代码段中的偏移地址

    ```c
    main(){
	    printf("%x\n",main);
    }
    ```
    思考：为什么这个程序能够打印出main函数在代码段中的偏移地址？
    ```bash
    1fa
    ```
    > 函数名作为表达式，经历函数到指针隐式转换，变成指向函数的指针值，转换后的指针值即函数的入口地址。

4. 用Debug加载ur1.exe，根据上面打印出的main函数的偏移地址，用u命令察看main函数的汇编代码。    
    仔细找到ur1.c中每条C语句对应的汇编代码。    
    注意：在这里，对于main函数汇编代码开始处的“push bp mov bp,sp”和结尾处的“pop bp”，这里只了解到：  
    这是C编译器安排的为函数中可能使用到bp寄存器而设置的，就可以了。
    - 生成的代码有一些变动，类似于：
    ```x86asm
    ;_AX=_BX+_CX  
    ; ==>
    MOV AX,BX   
    ADD AX,CX
    ```  
    ```x86asm
    PUSH BP
    MOV	BP,SP
    MOV	AX,0001
    MOV	BX,0001
    MOV	CX,0002
    MOV	AX,BX
    ADD	AX,CX
    MOV	AH,BL
    ADD	AH,CL
    MOV	AL,BH
    ADD	AL,CH
    POP	BP
    RET
    ```

5. 通过main函数后面有ret指令，我们可以设想：C语言将函数实现为汇编语言中的子程序。  
    研究下面程序的汇编代码，验证我们的设想。

    ```c
    void f(void);

    main(){
        _AX=1;_BX=1;_CX=2;
        f();
    }

    void f(void){
        _AX=_BX+_CX;
    }
    ```
    编译、连接后用Debug查看CS:01FA处汇编代码如下,容易验证设想正确
    ```x86asm
    PUSH BP
    MOV BP,SP
    MOV AX,0001
    MOV BX,0001
    MOV CX,0002
    CALL 020B
    POP BP
    RET
    PUSH BP ;CS:020B
    MOV BP,SP
    MOV AX,BX
    ADD AX,CX
    POP BP
    RET
    ```

## 研究试验 3 使用内存空间

1. 编一个程序um1.c  
    把um1.c保存在C:\minic下，编译，连接生成um1.exe。  
    然后用Debug加载um1.exe，对main函数的汇编代码进行分析，找到每条C语句对应的汇编代码；  
    对main函数进行单步跟踪，察看相关内存单元的内容。
    ```c
    main(){
        *(char *)0x2000='a';
        //这里int是2字节，即1字长
        *(int *)0x2000=0xf;
        //这里使用ES作为段地址寄存器
        //BX存放偏移地址
        *(char far *)0x20001000='a';

        _AX=0x2000;
        *(char *)_AX='b';

        _BX=0x1000;
        *(char *)(_BX+_BX)='a';
        //这里首先取右操作数入栈保存
        //再以CX放高位2000h BX放低位1000h作ADC加法
        //再将CX内容作为ES段地址寄存器的值访问2000:3000
        *(char far *)(0x20001000+_BX)=*(char *)_AX;
    }
    ```

2. 编一个程序，用一条C语句实现在屏幕的中间显示一个绿色的字符"a"
    ```c
    main(){
        *(int far *)(0xb8000000+160*12+2*40)=0x0261;
    }
    ```

3. 分析下面程序中所有函数的汇编代码，思考相关的问题  
    问题：C语言将全局变量存放在哪里？将局部变量存放在哪里？每个函数开头的"push bp mov bp,sp"有何含义？
    ```c
    int a1,a2,a3;

    void f(void);

    main(){
        int b1,b2,b3;
        //[01A6] [01A8] [01AA]
        a1=0xa1;a2=0xa2;a3=0xa3;
        //[BP-06] [BP-04] [BP-02]
        b1=0xb1;b2=0xb2;b3=0xb3;
    }

    void f(void){
        int c1,c2,c3;
        //[01A6]   [01A8]    [01AA]
        a1=0x0fa1;a2=0x0fa2;a3=0x0fa3;
        //[BP-06] [BP-04] [BP-02]
        c1=0xc1;c2=0xc2;c3=0xc3;
    }
    ```
    容易看出，全局变量保存在DS段中，而局部变量保存在SS段中  
    函数开头的"push bp mov bp,sp"意在保护现场，因为在代码中是直接更改SP的值来获得局部变量所需的空间(从这里也可以看出不会对局部变量进行值初始化)，再通过BP进行寻址。因此将旧的BP入栈，旧的SP值赋给BP。

4. 分析下面程序的汇编代码，思考相关的问题  
    问题：C语言将函数的返回值存放在哪里？
    ```c
    int f(void);

    int a,b,ab;

    main(){
        int c;
        c=f();
    }

    int f(void){
        ab=a+b;

        return ab;
    }
    ```
    从这里可以看出两点
    - 此处返回的值存放在寄存器AX中
    - 对全局变量进行了值初始化

5. 下面的程序向安全的内存空间写入从"a"到"h"8个字符，理解程序的含义，深入理解相关的知识。  
    （注意：请自己学习、研究malloc函数的用法）
    ```c
    #define Buffer ((char *)*(int far *)0x200)

    main(){
        Buffer=(char *)malloc(20);
        Buffer[10]=0;
        while(Buffer[10]!=8){
            Buffer[Buffer[10]]='a'+Buffer[10];
            Buffer[10]++;
        }
        free(Buffer);
    }
    ```
    - 这里宏被解释为将指向0000:0200处的 int * 类型的指针显示转化为char *再进行解引用。
    - 而malloc函数获取的20个字节长的内存块首地址，即返回的void *类型的指针被显示转化为char *类型的指针后存入Buffer所指向的内存空间。  
    - 这里malloc返回的值同样是经过AX来传递的，之后的代码就不用解释啦  
    - 最后博主机器上获取的内存块首地址250h通过栈作为参数传递给free函数，返回后pop给了CX,不知道干啥去了
    - 值得注意的是，从代码可以看出，malloc获得的内存在DS段中，所以并不会随着程序返回而释放，所以需要调用free函数手动释放。当然整个程序退出后随着DS段被释放，这片内存就不会泄露了

## 研究试验 4 不用main函数编程

- 程序f.c
    ```
    f(){
        *(char far *)(0xb8000000+160*10+80)='a';
        *(char far *)(0xb8000000+160*10+81)=2;
    }
    ```

1. 把程序f.c保存在C:\minic下，对其进行编译，连接，思考相关的问题
    - 编译和连接哪个环节会出问题？  
        链接（突然感觉这本书的用词都是连接，感觉链接用词更好
    - 显示出的错误信息是什么？  
        "Linker Error:Undefined Symbol '_Main' in Module C0S"
    - 这个错误信息可能和哪个文件有关？  
        C0S.OBJ

2. 用学习汇编语言时使用的 link.exe 对 tc.exe 生成的 f.obj 文件进行连接，生成f.  exe。  
    用Debug加载f.exe，察看整个程序的汇编代码。思考相关的问题
    - f.exe的程序代码总共有多少字节？  
        0-1Ch总共1Dh个字节
    - f.exe的程序能正确返回吗？
        可以显示绿色'a'，但是不能正确返回
    - f函数的偏移地址是多少？
        0000h

3. 写一个程序m.c  
    用tc.exe对m.c进行编译，连接，生成m.exe，用Debug察看m.exe整个程序的汇编代码。思考相关的问题
    ```c
    main(){
        *(char far *)(0xb8000000+160*10+80)='a';
        *(char far *)(0xb8000000+160*10+81)=2;
    }
    ```
    - m.exe的程序代码总共有多少字节？  
       根据DS=0ACEh,CS=0A77h推测可能是0057h字节(0ACEh-0A77h=0057h),但是这和main函数偏移地址CS:01FA相矛盾。不太清楚总共有多少字节的代码
    - m.exe的程序能正确返回吗？  
        可以
    - m.exe程序中的main函数和f.exe中的f函数的汇编代码有何不同？
        没有差别
    
4. 用Debug对m.exe进行跟踪：  
    注意：使用g命令和p命令。
    - 找到对main函数进行调用的指令的地址  
        和[研究试验 2 ](#toc-heading-3)中第2问一样，在CS:011A处对main函数进行了调用
    - 找到整个程序返回的指令  
        位于CS:0151-CS:0156处的3调指令，调用了int 21h中断例程

5. 思考如下几个问题
    - 对main函数调用的指令和程序返回的指令是哪里来的？
    - 没有main函数时，出现的错误信息里有和"C0S"相关的信息；而前面在搭建开发环境时，没有C0S.OBJ文件tc.exe就无法对程序进行连接。是不是tc.exe把C0S.OBJ和用户程序的.OBJ一起进行连接生成.exe文件？
    - 对用户程序的main函数进行调用的指令和程序返回的指令是否就来自C0S.OBJ文件？
    - 我们如何看到C0S.OBJ文件中的程序代码呢？
    - C0S.OBJ文件里有我们设想的代码吗？

    从报错的信息来推断，相关的代码应该来自C0S.OBJ，利用link.exe对其进行连接应该就可以了

6. 用link.exe对C:\minic目录下的C0S.OBJ进行连接，生成C0S.exe。用Debug分别察看C0S.exe和m.exe的汇编代码。  
    注意：从头开始察看，两个文件中的程序代码有和相同之处？
    - 报了6个L2029错误"unresolved external"
    - 总的来说差别不大

7. 用Debug找到m.exe中调用main函数的CALL指令的偏移地址，从这个偏移地址开始向后察看10条指令；  
    然后用Debug加载C0S.exe，从相同的偏移地址开始向后察看10条指令，对两处的指令进行对比。
    - 总体来说是一样的，几处偏移地址不太一样

8. 下面，我们用汇编语言编一个程序C0S.ASM，然后把它编译为C0S.OBJ，替代C:\minic目录下的C0S.OBJ。
    ```x86asm
    assume cs:code

    data segment
        db 128 dup (0)
    data ends

    code segment

    start:
        mov ax,data
        mov ds,ax
        mov ss,ax
        mov sp,128

        call s

        mov ax,4c00h
        int 21h

    s:

    code ends

    end start
    ```
    - 这里DS段从data segment的低地址向高地址增长，SS段从data segment的高地址向低地址增长  
    和PC机的内存划分有异曲同工之妙（最后还是会撞在一起233

9. 在C:\minic目录下，用tc.exe将f.c重新进行编译，连接，生成f.exe。  
    这次能通过连接吗？f.exe可以正确运行吗？用Debug察看f.exe的汇编代码。
    ```x86asm
    ;下面所有数字都是十六进制
    mov ax,0A77
    mov ds,ax
    mov ss,ax
    mov sp,0080

    call 0012

    mov ax,4C00
    int 21

    push bp ;CS:0012
    mov bp,sp
    mov bx,B800
    mov es,bx
    mov bx,0690
    es:
    mov byte ptr [bx],61
    mov bx,B800
    mov es,bx
    mov bx,0691
    es:
    mov byte ptr [bx],02
    pop bp
    ret
    ```
    - 可以正确链接、运行，程序显示绿色字符'a'，正常返回

10. 在新的C0S.OBJ的基础上，写一个新的f.c，向安全的内存空间写入从"a"到"h"8个字符，分析、理解f.c
    ```c
    #define Buffer ((char *)*(int far *)0x200)

    f(){
        Buffer=0;
        Buffer[10]=0;
        while(Buffer[10]!=8){
            Buffer[Buffer[10]]='a'+Buffer[10];
            Buffer[10]++;
        }
    }
    ```
    - 其实就是由通过malloc申请内存改为了使用C0S.ASM中data segment内DS段的内存空间，其他相关的程序编译器翻译的没什么差别

## 研究试验 5 函数如何接收不定数量的参数

用C:\minic下的tc.exe完成下面的试验  
1. 写一个程序a.c
    用tc.exe对a.c进行编译，连接，生成a.exe。用Debug加载a.exe，对函数的汇编代码进行分析。  
    解答这两个问题：main函数是如何给showchar传递参数的？showchar是如何接受参数的？
    ```c
    void showchar(char a,int b);

    main(){
        showchar('a', 2);
    }

    void showchar(char a,int b){
        *(char far *)(0xb8000000+160*10+80)=a;
        *(char far *)(0xb8000000+160*10+81)=b;
    }
    ```
    - main函数通过栈操作传递参数给showchar，参数自右向左入栈
    - showchar通过栈接收参数，但是不进行弹出，而是直接通过BP读取SS段中的内容  
    - 因为在参数入栈之后还进行了IP和BP的入栈操作  
        所以取参数char a时，加上idata 4，取参数int b时，加上idata 6

2. 写一个程序b.c  
    分析程序b.c，深入理解相关的知识。  
    思考：showchar函数是如何知道要显示多少个字符的？printf函数是如何知道有多少个参数的？
    ```c
    void showchar(int,int,...);

    main(){
        showchar(8,2,'a','b','c','d','e','f','g','h');
    }

    void showchar(int n,int color,...){
        int a;
        for(a=0;a!=n;a++){
            *(char far *)(0xb8000000+160*10+80+a+a)=*(int *)(_BP+8+a+a);
            *(char far *)(0xb8000000+160*10+81+a+a)=color;
        }
    }
    ```
    - showchar通过参数int n传递需要显示多少个字符，而参数n在最后入栈，SS段中BP+4即可获得
    - 参数从右向左入栈，最后入栈的参数是指向format首地址(偏移地址)的指针，而printf函数会调用相关函数对format进行解析。
        所以说printf的变参要和format中的占位符一一对应，否则会出现难以理解的输出。
        
3. 实现一个简单的printf函数，只需支持"%c %d"即可
    ```c
    MyPrintf(char * format,...);

    main(){
        MyPrintf("This is MyPrintf %c %d",'!',233);
    }

    MyPrintf(char * format,...){
        int cnt=0;
        int offset=0;
        while(*format!=0){
            switch (*format)
            {
            case '%':
                format+=1;
                if(*format=='c'){
                    *(char far *)(0xb8000000+160*12+40+2*offset++)=*(int *)(_BP+6+2*cnt++);
                }else if(*format=='d'){
                    int num=*(int *)(_BP+6+2*cnt++);
                    int len=0;
                    char tmp[10];
                    while (num!=0){
                        tmp[len++]=(num%10)+0x30;
                        num/=10;
                    }
                    len--;
                    while (len>=0){
                        *(char far *)(0xb8000000+160*12+40+2*offset++)=tmp[len--];
                    }
                }else{
                    *(char far *)(0xb8000000+160*12+40+2*offset++)=*format;
                }
                format+=1;
                break;
            default :
                *(char far *)(0xb8000000+160*12+40+2*offset++)=*format;
                format+=1;
                break;
            }
        }
    }
    ```
    输出
    ```bash
    This is MyPrintf ! 233
    ```

## 尾声

- Cover To Cover，算是读完啦
- 本以为综合研究会很难，实际上更多的是阅读代码理解思想。虽然学过编译原理，但是都忘光光了(＠_＠;)
- 很多人看不上16位汇编，觉得这是老古董，但是对于一些刚学完C的同学，再去看这本书绝对是有受用之处。
- 这本书被赞誉位经典并不过分，有很多有意思的地方，等学到其他地方再回想起来定会拍案叫绝。
- 最大的乐趣莫过于体验了一把没有GUI时的IDE