---
title: 深入理解计算机系统 DataLab
top: false
cover: false
img: 'http://bucket.jiabi.tech/medias/featureimages/csapp_b.jpg'
toc: true
mathjax: true
date: 2020-04-21 01:05:48
password:
summary: CSAPP DataLab实验报告
tags:
    - CSAPP
    - LAB
categories:
    - 计算机基础
---
# CSAPP DataLab实验报告

## 前言

>  很久很久很久之前便听说CSAPP是本神书  
    趁着狗东搞活动入了，上学期期末开始看断断续续到前一个月才看完第一章写完练习  
    再说读CSAPP若是不做LAB便少了几分乐趣  
    看了看下载列表，一个月之前下载的材料，因为实训还有在医院呆了段时间，到昨天才开始动手做  ╯︿╰

- 先说下博主的环境
    ```bash
    $ cat /proc/version
    Linux version 4.4.0-18362-Microsoft (Microsoft@Microsoft.com) (gcc version 5.4.0 (GCC) ) 
    #476-Microsoft Fri Nov 01 16:53:00 PST 2019
    ```
- 从官网下载到的handout里，Makefile指定了编译参数-m32,而博主使用的又是64bit的WSL，因此会报如下错误
    ```bash
    fatal error: bits/libc-header-start.h: No such file or directory
    #include <bits/libc-header-start.h>
            ^~~~~~~~~~~~~~~~~~~~~~~~~~
    compilation terminated.
    ```
    所以这里需要安装相关的库  
    另外WSL不支持32位程序，所以用qemu-user-i386来跑32位程序
    ```bash
    $ sudo service binfmt-support start
    ```
- 自己下载材料的话  
    通过dlc检测是否符合编码规则  
    通过btest检测结果是否正确以及进行打分  
    像"Beat the Prof"、BDD是没有的
- 附赠了两辅助工具ishow，fshow，虽然我觉得微软自带的计算器比ishow好用（逃
- 总共是13个谜题，因为在家挺作业中已经接触过类似的，总体感觉没有想象中的难  
    （还是有几题参考了其他人的代码  
- 有几题和书本上的家庭作业是重复的，看官网的意思"chosen from a set of 73 standard puzzles"  
    重复也是正常的作者可能认为那几个比较有价值  
- 建议把writeup和readme以及bits.c的介绍看完再解题，因为有一些限定  
    比如：因为dlc编写的原因，变量声明得在最前面，否则会报parser error
- 最后是博主的结果
    ```
    $ ./dlc -e bits.c
    dlc:bits.c:147:bitXor: 8 operators
    dlc:bits.c:157:tmin: 1 operators
    dlc:bits.c:170:isTmax: 7 operators
    dlc:bits.c:182:allOddBits: 9 operators
    dlc:bits.c:192:negate: 2 operators
    dlc:bits.c:207:isAsciiDigit: 10 operators
    dlc:bits.c:221:conditional: 7 operators
    dlc:bits.c:244:isLessOrEqual: 15 operators
    dlc:bits.c:256:logicalNeg: 5 operators
    dlc:bits.c:288:howManyBits: 36 operators
    dlc:bits.c:310:floatScale2: 16 operators
    dlc:bits.c:346:floatFloat2Int: 15 operators
    dlc:bits.c:366:floatPower2: 10 operators
    $ ./btest -g
    Score   Rating  Errors  Function
     1      1       0       bitXor
     1      1       0       tmin
     1      1       0       isTmax
     2      2       0       allOddBits
     2      2       0       negate
     3      3       0       isAsciiDigit
     3      3       0       conditional
     3      3       0       isLessOrEqual
     4      4       0       logicalNeg
     4      4       0       howManyBits
     4      4       0       floatScale2
     4      4       0       floatFloat2Int
     4      4       0       floatPower2
    Total points: 36/36
    ```

## bitXor

- 使用~ &两个运算符实现异或
- 如果了解一些布尔运算的话应该是很简单的  
    只是博主早把这些忘完了，也不知道当时怎么推出来的:sweat_smile:  
- ~~编写这篇博文时遇到了个Bug，后面直接跟着代码块会导致emoji无法显示~~
```c
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
    return ~(~(~x&y)&~(x&~y));
}
```

## tmin

- 返回补码整数能表示的最小值  
    这里假设int类型是32位的
```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
    // Refer:You may assume...32-bit representations of integers.
    return 1<<31;
}
```

## isTmax

- 判断是否是补码整数能表示的最大值  
    这里原先博主的做法达到了允许的Max ops  
    也就是所谓的"egregiously inefficient solutions"（惨  
    后来参考了其他人的代码作了调整
> This limit is very generous and is designed only to catch egregiously inefficient solutions.
```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
    // 10ops
    // return !(((x+1)&x)|(~((x+1)|x))|(!~x));
    return !(~(x+1+x) + !(x+1));
}
```

## allOddBits

- 这道题博主陷入了允许使用的最大constants是255的囚牢，尝试不使用掩码进行解题  
    尝试无果，股沟之，看了大家的解决方案还是用掩码，但是通过移位堆砌破除了这个255的限制  
    有了掩码就很好解决了
```c
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
    int mask = (0xAA<<24) + (0xAA<<16) + (0xAA<<8) + (0xAA);
    return !((mask&x)^mask);
}
```

## negate

- 取补码相反数，算是必知必会吧
```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
    return ~x+1;
}
```

## isAsciiDigit

- 判断是否在0x30-0x39之间  
    这里博主是通过观察特征做的判断，十分笨拙，建议忽略  
    参考了其他人的代码，那才是真正的位操作！
```c
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
    int high = (x^0x30)>>4;
    int low =  (x>>3)&((x>>2)|(x>>1))&0x01;
    return !(high|low);
}
```

## conditional

- 用位操作实现C的三目运算符 ? :  
    通过x构造出一个全0或全1的掩码就可以了
```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
    // int mask = ~!x+1;
    // avoid warning
    int mask = !x;
    mask = ~mask+1;
    return (~mask&y)+(mask&z);
}
```

## isLessOrEqual

- 通过位操作实现补码数比较的操作  
    这里不能用减法，可以通过位操作取补码相反数用加法实现减法，这样的话剩下的事情就是根据编码规则进行判断力  
    - x、y异号，x为负数，返回1
    - x、y同号，x+(-y)为负数或0，返回1
    - 其他情况返回0
```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
    // 0xFFFFFFFF or 0x00000000
    int difSign = (x^y)>>31;
    // x is negative and y is positive
    // then x must be less than y
    // 0x00000001 or 0x00000000
    int difSign_NegX = ((difSign&x)>>31)&0x1;

    // x and y are both positive
    int negY = ~y+1;
    int xSubY = x+negY;
    int resultSign = (xSubY>>31)&0x1;
    int isLessOrEqual = (resultSign|!xSubY) & !difSign;

    return difSign_NegX|isLessOrEqual;
}
```

## logicalNeg

- 使用位操作实现 !   
    这题博主参考了其他人的代码  
    思想是和相反数进行按位或操作，留下符号位进行算术右移
```c
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x) {
    return (((~x+1)|x)>>31)+1;
}
```

## howManyBits

- 这题的思路和家庭作业2.65odd_ones异曲同工  
    博主参考了其他人的代码，只能说amazing！
```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
    int sign = x>>31;
    int bit16,bit8,bit4,bit2,bit1,bit0;

    x = (sign&~x)|(~sign&x);

    bit16 = !!(x>>16)<<4;
    x = x>>bit16;
    bit8 = !!(x>>8)<<3;
    x = x>>bit8;
    bit4 = !!(x>>4)<<2;
    x = x>>bit4;
    bit2 = !!(x>>2)<<1;
    x = x>>bit2;
    bit1 = !!(x>>1);
    x = x>>bit1;
    bit0 = x;

    return bit16+bit8+bit4+bit2+bit1+bit0+1;
}
```

## floatScale2

- 计算2*f  
    其中f是unsigned表示的，刚开始没看到参数是uf，博主还奇怪不用cast怎么操作
    这题和家庭作业2.94float_twice是一样的  
    注意一下特殊情况，根据浮点数编码规则，仔细一点就可以了
```c
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf) {
    int sign = uf>>31<<31;
    int exp = uf>>23 & 0xff;

    if (exp==255) return uf;
    if (exp==0) return sign|uf<<1;
    exp++;
    if (exp==255) return sign|exp<<23;
    return sign|exp<<23|(0x807fffff&uf);
}
```

## floatFloat2Int

- 将unsigned表示的float转换为int  
    这题和家庭作业2.96float_f2i是一样的  
    注意一下特殊情况，根据浮点数和补码数编码规则，仔细一点就可以了
```c
/* 
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf) {
    int sign = uf>>31;
    int exp = uf>>23 & 0xFF;
    int frac = uf & 0x7FFFFF;
    int bias = 127;

    int x;

    if (exp < bias){
        x = 0;
    }else if(exp >= 31+bias){
        x = 0x80000000;
    }else{
        int E = exp - bias;
        int M = frac | 0x800000;
        if (E > 23) {
            x = M << (E - 23);
        } else {
            x = M >> (23 - E);
        }
    }

    return sign ? -x : x;
}
```

## floatPower2

- 使用浮点编码规则表示2^x  
    注意一下特殊情况，根据浮点数编码规则，仔细一点就可以了
```c
/* 
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 * 
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while 
 *   Max ops: 30 
 *   Rating: 4
 */
unsigned floatPower2(int x) {
    int bias = 127;
    if(x<-150) return 0;
    if(x>127) return 0xff<<23;
    if(x<-127) return 0x1<<(150-x);
    return (x+bias)<<23;
}
```

## 感想 

- 总体做下来感觉虽然没有想象中的难，但还是花了挺多时间  
    特别是像howManyBits根本就没有头绪  
    也惊叹于isAsciiDigit的解法，领略了位操作的神奇  
- 看过2015 CMU 15-213的第一节视频，作者说DataLab不允许迟交  
    因为这是最简单的，这个迟交了可能会导致用完宽限期  
    不知道后面那几个Lab博主还做不做得动 :worried: