---
title: C语言测量运行时间
author: lixinghui
date: 2023-2-23 12:00:00 +0800
categories: [C语言, Function]
tags: [C语言]
---



问：C语言，mingw编译器测试程序运行时间

## clock() 函数

要测试 C 语言程序的运行时间，可以使用 C 语言标准库中的 time.h 头文件中的 clock() 函数。

具体实现步骤如下：

1. 在程序的开头处，包含 time.h 头文件。

```c
#include <time.h>
```

1. 在需要测试的代码段的前后分别调用 clock() 函数，记录时间并计算程序运行时间。

```c
clock_t start_time, end_time;
double total_time;

start_time = clock(); // 记录起始时间

// 程序代码段

end_time = clock(); // 记录结束时间
total_time = (double)(end_time - start_time) / CLOCKS_PER_SEC;

printf("程序运行时间为 %lf 秒", total_time);
```

其中，start_time 和 end_time 变量记录了开始和结束时钟时间，total_time 变量保存了程序的运行时间，单位为秒。

需要注意的是，clock() 函数返回的是程序自启动以来所使用的 CPU 时钟周期数，而不是实际的时间。因此需要将时钟周期数除以 CLOCKS_PER_SEC，得到程序运行的实际时间。

另外，使用 Mingw 编译器时，需要在编译命令中加入 -O2 优化选项，以提高程序的运行速度。

下面是一个简单的示例程序，演示如何测试程序运行时间：

```c
#include <stdio.h>
#include <time.h>

int main()
{
    clock_t start_time, end_time;
    double total_time;

    start_time = clock();

    // 程序代码段
    int sum = 0;
    for (int i = 1; i <= 1000000; i++) {
        sum += i;
    }

    end_time = clock();
    total_time = (double)(end_time - start_time) / CLOCKS_PER_SEC;

    printf("程序运行时间为 %lf 秒", total_time);

    return 0;
}
```

运行结果如下：

```
程序运行时间为 0.000063 秒
```

---

问：能不能记录系统tick的变化

## GetTickCount()函数

可以通过获取系统tick的方式来记录系统tick的变化。在 Windows 操作系统中，系统tick是由系统计时器（system timer）以一定的时间间隔更新的计数器，可以通过 Win32 API 函数 GetTickCount() 或 GetTickCount64() 来获取当前系统tick的值。

具体实现步骤如下：

1. 包含 Windows.h 头文件。

```c
#include <Windows.h>
```

1. 调用 GetTickCount() 或 GetTickCount64() 函数，获取当前系统tick的值。例如，可以在程序开头处获取起始tick，然后在程序的某个关键点再次获取当前tick，两者相减就可以得到程序的运行时间。

```c
DWORD start_tick, end_tick, elapsed_ticks;

start_tick = GetTickCount();

// 程序代码段

end_tick = GetTickCount();
elapsed_ticks = end_tick - start_tick;

printf("程序运行时间为 %u 毫秒", elapsed_ticks);
```

其中，start_tick 和 end_tick 变量记录了起始和结束的系统tick，elapsed_ticks 变量保存了程序运行的tick数。

需要注意的是，GetTickCount() 函数返回的是自系统启动以来经过的毫秒数，如果运行时间比较长，可能会发生 DWORD 类型的溢出。此时可以使用 GetTickCount64() 函数，它返回的是自系统启动以来经过的毫秒数的 64 位无符号整数。

下面是一个简单的示例程序，演示如何记录系统tick的变化：

```c
#include <stdio.h>
#include <Windows.h>

int main()
{
    DWORD start_tick, end_tick, elapsed_ticks;

    start_tick = GetTickCount();

    // 程序代码段
    int sum = 0;
    for (int i = 1; i <= 1000000; i++) {
        sum += i;
    }

    end_tick = GetTickCount();
    elapsed_ticks = end_tick - start_tick;

    printf("程序运行时间为 %u 毫秒", elapsed_ticks);

    return 0;
}
```

运行结果如下：

```
程序运行时间为 15 毫秒
```

---

## QueryPerformanceCounter()函数

在Windows系统上，使用`clock()`函数测量函数运行时间时可能会出现精度问题。这是因为`clock()`函数在Windows系统上的精度通常较低，只有15.6ms，而且它只能精确到时钟周期的整数倍，因此在某些情况下可能会出现测量结果为0的情况。

为了解决这个问题，你可以使用Windows API中的`QueryPerformanceCounter()`和`QueryPerformanceFrequency()`函数来测量函数的运行时间。`QueryPerformanceCounter()`函数可以获取高精度计时器的当前计数值，`QueryPerformanceFrequency()`函数可以获取计时器的频率。通过这两个函数，可以计算出函数的运行时间。

以下是一个示例代码：

```c
#include <stdio.h>
#include <windows.h>

void fun_a() {
    // 在这里编写需要测试的代码
}

int main() {
    LARGE_INTEGER start_time, end_time, frequency;
    double duration;

    QueryPerformanceFrequency(&frequency);
    QueryPerformanceCounter(&start_time);
    fun_a();
    QueryPerformanceCounter(&end_time);

    duration = (double)(end_time.QuadPart - start_time.QuadPart) / frequency.QuadPart;
    printf("fun_a() takes %f seconds to run.\n", duration);

    return 0;
}
```

在上面的代码中，`LARGE_INTEGER`是Windows API中定义的一个结构体，用来保存64位整数值。`QueryPerformanceFrequency()`函数用来获取计时器的频率，`QueryPerformanceCounter()`函数用来获取当前计数值。`duration`变量用来保存函数运行的时间，计算方法是当前计数值之差除以计时器频率。

需要注意的是，`QueryPerformanceCounter()`和`QueryPerformanceFrequency()`函数是Windows API中的函数，不是标准C语言库中的函数，因此需要包含`windows.h`头文件才能使用。

其中，在windows中通过mingw编译器，`frequency.QuadPart:10000000`，也就是`QueryPerformanceCounter()`这个函数获取的结果是以`0.1us`为单位的。
