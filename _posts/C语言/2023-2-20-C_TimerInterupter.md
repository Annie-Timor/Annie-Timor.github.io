---
title: C语言定时器中断
author: lixinghui
date: 2023-2-20 12:00:00 +0800
categories: [C语言, Timer]
tags: [C语言]
---


**前提提要：**

> 电脑端运行的系统不同于单片机运行的系统，电脑端为软实时系统，对于中断的到来，可能会受到其他进程或线程的调度所影响，从而CPU无法及时响应，所以电脑端的定时器仅做模拟作用。

## SetTimer触发定时器

```c
#include <stdio.h>
#include <windows.h>

void CALLBACK timer_callback(HWND hwnd, UINT uMsg, UINT_PTR idEvent, DWORD dwTime)
{
    // 中断回调函数
    // 在这里实现每秒钟执行的操作
    printf("Timer callback\n");
}

int main()
{
    UINT_PTR timerid = SetTimer(NULL, 0, 1000, timer_callback);

    // 在这里执行其他操作

    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0))
    {
        printf("msg??\n");
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0;
}

```

`SetTimer()` 是 Windows API 中的一个函数，用于创建一个计时器，让程序在指定的时间间隔内重复执行指定的回调函数。

该函数的声明如下：

```c
UINT_PTR SetTimer(
  HWND      hWnd,
  UINT_PTR  nIDEvent,
  UINT      uElapse,
  TIMERPROC lpTimerFunc
);
```

各个参数的含义如下：

1. `hWnd`: 与计时器相关联的窗口句柄。如果计时器与任何窗口无关，则此参数应为NULL。
2. `nIDEvent`: 计时器ID。如果计时器ID为0，则系统将为计时器分配一个唯一的ID。
3. `uElapse`: 计时器时间间隔，以毫秒为单位。每隔 `uElapse` 毫秒后，系统会触发计时器事件，调用回调函数。
4. `lpTimerFunc`: 回调函数指针。当计时器事件触发时，系统将调用此函数。

该函数返回一个 `UINT_PTR` 类型的值，表示计时器ID。如果调用失败，则返回值为0。

需要注意的是，Windows API 中的计时器时间间隔的最小值为15毫秒。如果需要更高的精度，则可以使用 `timeSetEvent()` 函数创建高精度定时器。

:loudspeaker::以上这个示例代码中，`SetTimer` 函数用来创建一个定时器，其中 `TimerProc` 是定时器回调函数，每当定时器事件被触发时都会调用它。在主函数中，使用 `GetMessage` 函数等待消息队列中的消息，并使用 `TranslateMessage` 函数和 `DispatchMessage` 函数处理消息。由于 `GetMessage` 函数是一个阻塞函数，它会等待消息到来，因此可以保证程序不会退出。最后使用 `KillTimer` 函数来停止定时器。



## timeSetEvent触发定时器

timeSetEvent`()`函数需要包含头文件`#include <mmsystem.h>`，该文件为windows平台SDK的一个头文件，估计是没有正确的安装Windows平台SDK，无法体验到。

## CreateTimerQueueTimer创建定时器队列

`CreateTimerQueueTimer` 函数是用于创建一个定时器的 Windows API 函数。它的参数如下：

```c
BOOL CreateTimerQueueTimer(
  PHANDLE                phNewTimer,
  HANDLE                 TimerQueue,
  WAITORTIMERCALLBACK    Callback,
  PVOID                  Parameter,
  DWORD                  DueTime,
  DWORD                  Period,
  ULONG                  Flags
);
```

下面是各个参数的详细解释：

1. `phNewTimer`：一个指向 `HANDLE` 类型的指针，用于接收创建的定时器的句柄。定时器句柄用于标识定时器，可以用于删除定时器。
2. `TimerQueue`：一个 `HANDLE` 类型的句柄，指定定时器所属的定时器队列。定时器队列是一个系统级别的对象，用于管理多个定时器的触发和回调函数的执行。
3. `Callback`：一个指向回调函数的指针，用于指定定时器事件发生时要执行的函数。回调函数的参数类型为 `PVOID`。
4. `Parameter`：一个指向任意数据的指针，用于向回调函数传递参数。
5. `DueTime`：一个 `DWORD` 类型的值，指定定时器首次触发的时间间隔（以毫秒为单位）。如果设置为 `0`，表示定时器立即触发。
6. `Period`：一个 `DWORD` 类型的值，指定定时器下一次触发的时间间隔（以毫秒为单位）。如果设置为 `0`，表示定时器只触发一次。
7. `Flags`：一个 `ULONG` 类型的值，表示定时器的行为标志。可以设置为 `WT_EXECUTEINTIMERTHREAD` 或 `WT_EXECUTEONLYONCE`。其中，`WT_EXECUTEINTIMERTHREAD` 表示在定时器线程上执行回调函数，`WT_EXECUTEONLYONCE` 表示定时器只触发一次。

需要注意的是，`CreateTimerQueueTimer` 函数创建的定时器只能用于在 Windows 线程池环境下工作，不能用于单线程或多线程环境。如果您的程序是单线程或多线程环境，请使用 `SetTimer` 函数或其他定时器库。

`Period` 参数指定了定时器的时间间隔，也就是每隔多少时间触发一次定时器。它的最小值是 0，表示定时器只会触发一次。如果设置为非 0 的值，表示定时器会每隔一定时间触发一次，直到定时器被取消为止。

`Period` 参数的取值范围是 0 到 `MAXDWORD`（4294967295）之间的任意值，单位为毫秒。需要注意的是，如果 `Period` 的值过小，会对系统性能产生影响，因此应该谨慎设置。一般来说，建议将 `Period` 设置为 100 毫秒以上，以免对系统性能产生过大影响。

## 示例代码：

```c
#include <stdio.h>
#include <windows.h>

HANDLE timer_queue = NULL;

void CALLBACK TimerCallback(PVOID lpParameter, BOOLEAN TimerOrWaitFired)
{
    printf("Timer event\n");
}

int main()
{
    // 创建定时器队列
    timer_queue = CreateTimerQueue();
    if (timer_queue == NULL)
    {
        printf("Failed to create timer queue\n");
        return 1;
    }

    // 创建定时器
    HANDLE timer_handle = NULL;
    if (!CreateTimerQueueTimer(&timer_handle, timer_queue, TimerCallback, NULL, 1000, 1000, 0))
    {
        printf("Failed to create timer\n");
        DeleteTimerQueue(timer_queue);
        return 1;
    }

    // 等待一段时间
    Sleep(5000);

    // 删除定时器
    if (!DeleteTimerQueueTimer(timer_queue, timer_handle, NULL))
    {
        printf("Failed to delete timer\n");
    }

    // 删除定时器队列
    if (!DeleteTimerQueue(timer_queue))
    {
        printf("Failed to delete timer queue\n");
    }

    return 0;
}

```

在这个示例代码中，首先使用 `CreateTimerQueue` 函数创建一个定时器队列，然后使用 `CreateTimerQueueTimer` 函数创建一个定时器，并在定时器回调函数中输出一条信息。接下来等待一段时间，最后删除定时器和定时器队列。

需要注意的是，`CreateTimerQueue` 函数和 `CreateTimerQueueTimer` 函数都可以返回一个 `BOOL` 类型的值，表示函数调用是否成功。在实际使用时，需要根据返回值来判断函数是否成功。同时，使用 `DeleteTimerQueueTimer` 函数删除定时器时需要指定定时器句柄，这个句柄是在调用 `CreateTimerQueueTimer` 函数时返回的。

另外，需要注意的是，定时器队列需要在程序退出前使用 `DeleteTimerQueue` 函数删除，否则会导致资源泄露。