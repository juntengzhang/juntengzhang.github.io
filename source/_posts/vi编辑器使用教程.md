---
title: vi编辑器使用教程
tags: linux
date: 2018-10-21 14:30:46
---


一开始用私人服务器，linux系统都是ubuntu，有Xwindow，所以编辑文件是直接用的gedit，把linux活生生用成了window，因为最近需要使用ssh远程，只有tty，改用vi，指令不熟练，稍微记录一下基本指令。
<!--more-->
vi有3种模式：插入模式、命令模式、低行模式。

命令模式：默认模式，在此模式下可以移动光标、进行删除、撤销、搜索、替换、复制、粘贴等操作。

插入模式：在此模式下可以输入字符，按ESC将回到命令模式。

低行模式：可以保存文件、退出vi等操作。

## 一、移动光标（命令模式）

1. 使用方向键
2. h 向左；j 向下；k 向上；l 向右；Enter 移动到下一行首；- 移动到上一行首

## 二、删除、撤销操作（命令模式）
x   //删除当前字符

nx  //删除从光标开始的n个字符

dd      //删除当前行

ndd     //向下删除当前行在内的n行

u       //撤销上一步操作

U       //撤销对当前行的所有操作

## 三、搜索（命令模式）
/zjt    //向光标下搜索zjt字符串

?zjt    //向光标上搜索zjt字符串

n       //向下搜索前一个搜素动作

N       //向上搜索前一个搜索动作


## 四、跳至指定行（命令模式）
n+      //向下跳n行

n-      //向上跳n行

nG  //跳到行号为n的行

G       //跳至文件的底部

## 五、复制、粘贴、替换（命令模式）
yy      //将当前行复制到缓存区

ayy     //将当前行复制到缓存区,a为缓冲区，可替换为a到z的任意字母，完成多个复制任务

nyy     //将当前行向下n行复制到缓冲区

anyy    //将当前行向下n行复制到缓冲区,a为缓冲区，可替换为a到z的任意字母，完成多个复制任务

yw      //复制从光标开始到词尾的字符

nyw     //复制从光标开始的n个单词

y^      //复制从光标到行首的内容

y$      //复制从光标到行尾的内容

p       //粘贴剪切板里的内容在光标后，若使用自定义缓冲区，使用ap进行粘贴

P       //粘贴剪切板里的内容在光标前，若使用自定义缓冲区，使用aP进行粘贴

:s/old/new          //用new替换行中首次出现的old

:s/old/new/g        //用new替换行中所有的old

:n,m s/old/new/g        //用new替换从n到m行里所有的old

:%s/old/new/g       //用new替换当前文件里所有的old

## 六、插入文本或行(命令模式下使用，进入插入模式)
a       //在当前光标位置的右边添加文本

i       //在当前光标位置的左边添加文本

A       //在当前行的末尾位置添加文本

I       //在当前行的开始处添加文本(非空字符的行首)

O       //在当前行的上面新建一行

o       //在当前行的下面新建一行。

R       //替换(覆盖)当前光标位置及后面的若干文本

J       //合并光标所在行及下一行为一行（仍为命令模式）

## 七、打开文件、保存、关闭文件（命令模式使用，进入低行模式）
:w      //保存文件

:q      //退出编辑器(若文件已修改则使用以下的命令)

:q!     //退出编辑器(不保存文件)

:wq     //退出编辑器(保存文件)

## 八、设置行号（命令行模式使用，进入低行模式）
:set  nu    //显示行号

:set nonu   //取消显示行号
