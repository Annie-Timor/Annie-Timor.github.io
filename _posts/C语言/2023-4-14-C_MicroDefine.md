---
title: C语言宏定义概述
author: lixinghui
date: 2023-4-14 12:00:00 +0800
categories: [C语言, Micro]
tags: [C语言]
---


宏定义又称为宏替换，在程序编译的时候对宏定义的内容进行替换。

正如C语言中所讲，函数的使用可以使程序更加模块化，便于组织，而且可重复利用，但在发生函数调用时，需要保留调用函数的现场，以便子函数执行结束后能返回继续执行，同样在子函数执行完后要恢复调用函数的现场，这都需要一定的时间，如果子函数执行的操作比较多，这种转换时间开销可以忽略，但如果子函数完成的功能比较少，甚至只完成一点操作，如一个乘法语句的操作，则这部分转换开销就相对较大了，但使用带参数的宏定义就不会出现这个问题，因为它是在预处理阶段即进行了宏展开，在执行时不需要转换，即在当地执行。宏定义可完成简单的操作，但复杂的操作还是要由函数调用来完成，而且宏定义所占用的目标代码空间相对较大。所以在使用时要依据具体情况来决定是否使用宏定义。

## 1、定义常量

-   `#define`指令用于定义常量
-   用大写字母表示宏定义，以便于和变量区分开来。

例如：

```c
#define MAX_VALUE 100
```

## 2、定义函数

-   宏定义的函数不能指定参数类型

例如：

```c
#define SQUARE(x) ((x) * (x))
#define MAX(a, b) ((a) > (b) ? (a) : (b))
```

## 3、位运算符

例如：

```c
#define BIT(x) (1 << (x))

int n = 5;
if ((n & BIT(2)) != 0) {
    printf("The 2nd bit of n is set\n");
}
```

## 4、计算数组的数量

例如：

```c
#define ARRAY_SIZE(arr) (sizeof(arr) / sizeof(arr[0]))

int arr[] = {1, 2, 3, 4, 5};
int size = ARRAY_SIZE(arr);   // size等于5
```

## 5、转字符串

-   `#XSTRING`运算符用于将宏参数转换为字符串常量；

例如：

```
#define STRINGIFY(x) #x
printf("%s\n", STRINGIFY(foo));   // 输出 "foo"
```

## 6、拼接

-   `##`运算符用于将两个标记合并为一个标记，并用于定义函数、数据结构等。

例如：

```c
#define CONCAT(x, y) x##y
int CONCAT(prefix, a) = 1;        // 定义变量 "prefixa" 并赋值为 1
```

## 7、取消定义

-   `#undef` 用于取消宏定义

例如：

```c
#define DEBUG
// ...
#undef DEBUG
```

上述代码中，首先定义了宏DEBUG，然后使用#undef指令取消宏定义。

## 8、条件编译

-   使用条件编译可以根据不同的情况编译不同的代码；
-   \#if、#elif、#else、#endif指令用于条件编译。

例如：

```c
#define MODE 2

#if MODE == 1
#define MAX_VALUE 50
#elif MODE == 2
#define MAX_VALUE 100
#else
#define MAX_VALUE 200
#endif

// ...
```

上述代码中，根据宏MODE的值来定义宏常量MAX_VALUE的值，并根据不同情况编译不同的代码。

## 9、使用可变参数宏

-   可变参数宏可以使用不定数量的参数；
-   \#define指令可以定义可变参数宏。

例如：

```c
#include <stdio.h>
#include <stdarg.h>

#define DEBUG_PRINT(...) debug_print(__FILE__, __LINE__, __VA_ARGS__)

void debug_print(const char* file, int line, const char* format, ...)
{
    va_list arg_ptr;
    va_start(arg_ptr, format);
    printf("%s:%d: ", file, line);
    vprintf(format, arg_ptr);
    printf("\n");
    va_end(arg_ptr);
}

int main()
{
    int a = 5;
    DEBUG_PRINT("a = %d", a);   // 输出 "main.c:XX: a = 5"
    return 0;
}
```

上述代码中，定义了可变参数宏DEBUG_PRINT，使用可变参数宏可以方便地在程序中输出调试信息。

## 10、使用静态断言

-   使用静态断言可以在编译阶段检查代码的正确性；
-   \#define指令可以定义静态断言。

例如：

```c
#define STATIC_ASSERT(expr) typedef char static_assertion[(expr) ? 1 : -1]

STATIC_ASSERT(sizeof(int) == 4);   // 检查int是否为4个字节
```

上述代码中，定义了静态断言宏STATIC_ASSERT，用于检查int是否为4个字节。如果检查结果为真，则重新声明一个char类型为数组，正确的话数组长度为1，否则数组长度为-1，在编译期间将会发生错误。

这些宏定义小妙招和技巧可以帮助我们更好地使用C语言，提高代码的可读性、可维护性和可扩展性。

## 11、宏定义中的do...while(0)

`do...while(0)`在宏定义中使用较多，原因为，在宏定义展开的时候不会出错。

在Linux和其它代码库里的，很多宏实现都使用do/while(0)来包裹他们的逻辑，这样不管在调用代码中怎么使用分号和大括号，而该宏总能确保其行为是一致的。



## 参考：

[C语言宏#define(精通详解) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/367761694)

[do{...}while(0)的妙用 - 简书 (jianshu.com)](https://www.jianshu.com/p/99efda8dfec9)

chatGPT

