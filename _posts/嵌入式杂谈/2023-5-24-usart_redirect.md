---
title: 串口重定向
author: lixinghui
date: 2023-5-24 12:00:00 +0800
categories: [mcu]
tags: [mcu]
---


在单片机的串口输出中，只能通过单字节发送或者字符串发送，这对调试的输出不太方便，如果我们能使用像C语言`printf`函数来输出就会方便很多。

标准的`stdio.h`中对于`printf`函数的输出是有固定方向（显示器）的，如果需要输出到串口，就需要对它进行重定向处理。

32位单片机的`printf`函数是调用了`fputc`函数，8位单片机为`putchar`函数。

重定向`fputc`函数代码为：

```c
#include <stdio.h>

int fputc(int ch, FILE *f)
{
  HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xffff);
  return ch;
}
```

重定向`putchar`函数代码为：

```c
#include <stdio.h>

char putchar(char c)
{
	SendByte(c);
	return c;
}
```



如果是多个串口都想要像`printf`一样的功能，需要包括头文件`#include <stdarg.h>`，代码如下：

```c
// 定义一个类似于printf的函数
void my_printf(const char *format, ...)
{
    char buffer[256]; // 用于存储格式化后的字符串

    // 使用标准C库函数vsnprintf将可变参数列表格式化为字符串，并存储到buffer中
    va_list args;
    va_start(args, format);
    vsnprintf(buffer, sizeof(buffer), format, args);
    va_end(args);

    // 调用HAL_UART_Transmit函数将buffer中的字符串发送出去
    HAL_UART_Transmit(&huart1, (uint8_t*)buffer, strlen(buffer), HAL_MAX_DELAY);
}
```

