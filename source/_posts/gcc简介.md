---
title: gcc简介
date: 2018-12-05 11:54:04
tags:
---
gcc即GNU编译器套件（GNU Compiler Collection）包括C、C++、Objective-C、Fortran、Java、Ada和Go语言的前端，也包括了这些语言的库（如libstdc++、libgcj等等）
<!--more-->
## 执行流程
用gcc由C语言源代码文件生成可执行文件的过程不仅仅是编译的过程，而是要经历四个相互关联的步骤∶预处理(也称预编译，Preprocessing)、编译(Compilation)、汇编(Assembly)和链接(Linking)
+ 示例代码
```
test.c
#include<stdio.h>

int main(void)
{
    printf("hello\n");
    return 0;
}
```

+ 预编译，.c -> .i
```
gcc -E test.c -o test.i
```

+ 编译，.i -> .s
```
gcc -S test.i -o test.s
```

+ 汇编，.s -> .o
```
gcc -c test.s -o test.o
```

+ 链接
```
gcc test.o -o test
```

## gcc常用选项
+ -O -O2 -O3
gcc对源码进行优化

+ -g 
告诉gcc产生能被GNU调试器（如gdb)使用的调试信息，以便调试用户的程序

+ -fPIC -shared
生成动态链接库.so，-shared表明产生共享库，而-fPIC(Position Independent Code)表明使用地址无关代码
```
gcc -fPIC -shared test.c -o libtest.so

or

gcc -fPIC -c test.c -o test.o
ld -shared test.o -o libtest.so
```
