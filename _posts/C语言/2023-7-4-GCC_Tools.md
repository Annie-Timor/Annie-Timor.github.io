---
title: GCC工具链
author: lixinghui
date: 2023-7-3 12:00:00 +0800
categories: [C语言, GCC]
tags: [C语言]
---




## 编译过程

1、预处理：处理所有以#开头的代码，包括头文件，宏定义，条件编译。

```
gcc -E main.c -o main.i
```

2、编译：语法检查将C语言代码变成汇编代码。

```
gcc -S main.i -o main.s
```

3、汇编：将汇编代码变成二进制文件。

```
gcc -c main.s -o main.o
```

4、链接：链接代码所需要的其他文件，包括库文件等等。

```
gcc main.o -o main
```



## 其他参数

```
-v / --version ：查看gcc版本号
-I目录 ：include指定头文件目录，注意-I和目录之间没有空格
-L库文件 ：library指定库文件目录
-l库 ：链接库文件，这里-lwinmm 为链接libwinmm的库文件
-g ：生成可执行文件包括调试信息，gdb调试一定需要调试信息
-On n=0~3 ：编译优化，n越大优化越多
-Wall ：提示更多警告信息 warning all
-static ：链接静态库

```



假设有4个文件，分别为`f1.c f2.c f.h main.c`

```c
// main.c
#include <stdio.h>
#include "f.h"

int main(int argc, char **argv)
{
    f1();
    f2();
    return 0;
}

// f1.c
#include <stdio.h>
#include "f.h"

void f1(void)
{
    printf("this is f1 function\n");
}

// f2.c
#include <stdio.h>
#include "f.h"

void f2(void)
{
    printf("this is f2 function\n");
}

//f.h
#ifndef _F_H
#define _F_H

void f1(void);
void f2(void);

#endif
```





## linux中静态库生成和使用

假如有两个源文件为`f1.c/f2.c`，一个头文件为`f.h`，现在需要将`f1.c/f2.c`的内容编译为静态库，命令如下：

```
gcc -c f1.c f2.c             #生成源文件的二进制文件
ar -crv libf f1.o f2.o       #将二进制文件链接生成库

也可以直接
ar -crv libf f1.c f2.c       #直接通过源文件生成库文件
```

>   `ar`调用静态库归档工具ar
>
>   ` -crv `创建一个新的归档文件，同时向其中添加指定的目标文件。具体含义如下：
>          `-c`：创建一个新的归档文件。
>          `-r`：向归档文件中添加指定的目标文件。
>          `-v`：显示详细的操作过程（verbose mode）。

到这里就生成了`libf`的静态库，其中`lib`是一定需要的，`f`才是这个库的名称。

**使用静态库**

```
gcc main.c -o a -static -L. -lf
```

>   `-static`表示静态链接
>
>   `-L.`表示指定库文件路径为当前目录
>
>   `-lf`表示链接的库文件为`f`
>
>   这个命令也可以写成：
>
>   ```
>   gcc main.c -static -L. -lf -o a
>   ```
>
>   效果都是一样的。

## linux中动态库的生成和使用

```
gcc -c f1.c f2.c
gcc -fPIC -shared -o libff.so f1.o f2.o

gcc -fPIC -shared -o libff.so f1.c f2.c
```

>   `-fPIC`生成位置无关代码（Position Independent Code）
>
>   `-shared`生成动态链接库

**使用动态库**

```
gcc main.c -o a -L. -lff
```

需要注意的是：gcc编译器链接动态库是去系统默认地方查找，通过命令`ldd a`可以看到系统默认查找路径，这里需要将生成的库假如到系统中去。

例如这里的系统默认为`/lib/x86_64-linux-gnu/libc.so.6 (0x00007fb4e42d2000)`

## windows中静态库的生成和使用

静态库的生成：

```
gcc -c f1.c f2.c
ar rc libf.a f1.o f2.o
```

静态库的使用：

```
gcc main.c -o a -L. -lf
```

windows和linux在静态库的生成近乎一样，只是linux中是`ar -crv`，而windows中是`ar rc`。

linux中生成的静态库没有后缀，windows中生成的静态库后缀为`.a`

## windows中动态库的生成和使用

生成：

```
gcc -shared -o libff.dll f1.c f2.c
```

使用:

```
gcc main.c -o a -L. -lff
```

在Windows中的动态库后缀为`.dll`文件，在linux中后缀为`.so`文件。

## windows系统中和linux系统中的索引方式区别

在 Windows 系统和 Linux 系统中，程序在运行时需要能够找到动态链接库才能正确运行。不同的操作系统在寻找动态链接库的方式上有所不同。

在 Windows 系统中，程序在运行时会首先搜索包含程序文件的目录，然后搜索系统 PATH 环境变量中列出的目录，以及注册表中列出的目录。因此，如果将动态链接库放在程序文件所在的目录或者系统 PATH 环境变量中列出的目录，程序就能够找到并使用这个动态链接库了。

而在 Linux 系统中，程序在运行时会搜索一系列默认的目录，其中包括 `/lib`、`/usr/lib`、`/usr/local/lib` 等。如果将动态链接库放在这些默认的目录中，程序就能够找到并使用这个动态链接库了。当然，也可以使用 `LD_LIBRARY_PATH` 环境变量或者修改 `/etc/ld.so.conf` 文件来指定其他的动态链接库搜索路径。

需要注意的是，Linux 系统中将自己生成的动态链接库拷贝到 `/lib/x86_64-linux-gnu/` 目录是因为这是系统默认的动态链接库路径之一。这样做的好处是可以让系统中的所有程序都能够找到并使用这个库，而不需要为每个程序单独设置搜索路径。当然，如果你想使用其他的动态链接库搜索路径，也可以在编译时使用 `-rpath` 选项或者在运行时使用 `LD_LIBRARY_PATH` 环境变量来指定。



参考：

[gcc工具链 ](https://www.bilibili.com/video/BV1ZT4y137Ba/?spm_id_from=333.337.search-card.all.click)

[gcc简介和命令行参数说明_gcc -t_KuoGavin的博客-CSDN博客](https://blog.csdn.net/yueguangmuyu/article/details/116703618)

chatGPT