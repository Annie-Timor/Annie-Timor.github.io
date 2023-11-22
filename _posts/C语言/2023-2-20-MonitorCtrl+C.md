---
title: C语言检测Ctrl+C触发
author: lixinghui
date: 2023-2-20 12:00:00 +0800
categories: [C语言, Function]
tags: [C语言]
---



​	在 Windows 的 C 语言中，我们可以使用 Windows API 中的 `SetConsoleCtrlHandler()` 函数来检测用户是否按下了 Ctrl+C 来终止程序。这个函数可以设置一个回调函数，在用户按下 Ctrl+C 时自动调用该回调函数。

```c
#include <stdio.h>
#include <windows.h>

BOOL g_bCtrlCPressed = FALSE;

BOOL WINAPI ConsoleCtrlHandler(DWORD dwCtrlType)
{
    if (dwCtrlType == CTRL_C_EVENT)
    {
        printf("Ctrl+C detected, cleaning up...\n");
        g_bCtrlCPressed = TRUE;
        return TRUE;
    }
    return FALSE;
}

int main()
{
    /*
        使用 SetConsoleCtrlHandler() 函数来设置控制事件处理程序，
        该函数的第一个参数为一个回调函数指针，第二个参数为一个布尔值，
        表示是否将当前程序作为处理程序添加到控制台。
        在回调函数中，可以执行相应的清理操作。
        SetConsoleCtrlHandler() 函数只对控制台应用程序有效
    */
    if (!SetConsoleCtrlHandler(ConsoleCtrlHandler, TRUE))
    {
        printf("Error: Could not set console control handler\n");
        return 1;
    }

    while (!g_bCtrlCPressed)
    {
        // Do some work here...
        printf(".");
        Sleep(100);
    }

    printf("Exiting...\n");
    return 0;
}

```

​	在上面的代码中，`SetConsoleCtrlHandler()` 函数用于设置一个回调函数 `console_ctrl_handler()`，该函数在用户按下 Ctrl+C 时被调用。在回调函数中，我们可以根据 `ctrl_type` 参数的值来判断用户按下了哪个键。如果用户按下了 Ctrl+C，那么我们可以在回调函数中输出一条信息，然后返回 `TRUE` 来表示已经处理了该事件。如果返回 `FALSE`，则表示事件未处理，系统会继续向下传递该事件。

​	在程序的主函数中，我们通过调用 `SetConsoleCtrlHandler()` 函数来注册回调函数。如果调用失败，则表示无法设置控制句柄，我们可以在标准错误流中输出一条错误消息。程序的其他操作可以在 `while` 循环中执行，当用户按下 Ctrl+C 后，回调函数会被调用，程序会在 `while` 循环中停止，然后退出。

​	需要注意的是，当用户按下 Ctrl+C 时，系统会向所有正在运行的线程发送一个中断信号，因此我们需要确保程序的其他线程也能够正确地响应这个信号，以避免程序出现异常行为。如果程序中有其他需要执行清理操作的资源（例如文件、套接字等），也需要在回调函数中进行适当的清理。
