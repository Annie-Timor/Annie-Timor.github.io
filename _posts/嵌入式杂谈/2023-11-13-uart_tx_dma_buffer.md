---
title: 一种基于串口DMA传输的缓冲区方案
author: lixinghui
date: 2023-11-13 12:00:00 +0800
categories: [mcu]
tags: [嵌入式杂谈]
---


在日常使用中，最常见的就是直接使用串口的寄存器输出，以HAL库为例，使用的函数为：

```c
HAL_StatusTypeDef HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout)
```

如果需要方便一点就可以重写`printf`函数的输出函数`fputc`，如下：

```c
#include <stdio.h>

int fputc(int ch, FILE *f)
{
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xffff);
    return ch;
}
```

但是以上的输出方式都是阻塞式输出，会卡住MCU的其他计算，而且串口输出的速度较慢，通过测量，下面为串口波特率为115200，输出52个字节的时间：

```
time1: calcTime 4.521070 ms, calcTicks 325517
time2: calcTime 4.983208 ms, calcTicks 358791
```

其中`time1`为调用系统函数`HAL_UART_Transmit`直接输出的，`time2`为使用`printf`函数进行输出的，通过计算在波特率为115200的时候，每秒钟输出的字节数为：$115200/(8+1+1)=11520$，其中为8位数据位，1位起始位，1位停止位，纯寄存器输出52个字节的时间为4.514ms，加上一些输出时候的寄存器配置，和测量得出的时间一致，`printf`因为需要重写底层的函数所用需要的时间长一点。

当输出字符的数量过多的时候，串口打印会影响主程序的运行，同时因为串口导致的错误在某些时候并不容易察觉，如另一个DMA通过循环的模式的输出的时候，所用如果能够去除掉串口发送占用时间的影响会好很多。

Talk is  cheap, Show me the code.

```c
#include <stdarg.h>
#include <string.h>

UART_HandleTypeDef *g_uart = &huart1; /*硬件外设取指针*/
DMA_HandleTypeDef *g_dma_tx = &hdma_usart1_tx;

#define X_SIZE (1024)                 /*缓冲区大小*/
char buffer_array[X_SIZE];            /*缓冲区数组*/
int index = 0;                        /*数组索引*/
int is_resend = 0;                    /*需要再次发送*/
int resend_len = 0;                   /*再次发送的长度*/
char *resend_addr = &buffer_array[0]; /*再次发送地址*/

int my_printf(const char *format, ...)
{
    char buffer[256] = {0};
    int len = 0;
    int ret = 0;

    // 使用标准C库函数vsnprintf将可变参数列表格式化为字符串，并存储到buffer中
    va_list args;
    va_start(args, format);
    len = vsnprintf(buffer, 256, format, args);
    va_end(args);

    /*判断是否正在发送数据*/
    if (g_dma_tx->State == HAL_DMA_STATE_READY)
    {
        memset(buffer_array, 0, sizeof(buffer_array));
        memcpy(buffer_array, buffer, len);
        index = len;

        resend_len = 0;
        resend_addr = &buffer_array[index];
        HAL_UART_Transmit_DMA(g_uart, (uint8_t *)buffer_array, len);
    }
    else if (g_dma_tx->State == HAL_DMA_STATE_BUSY)
    {
        if ((len <= X_SIZE - index))
        {
            /*可以直接放下*/
            memcpy(&buffer_array[index], buffer, len);
            is_resend = 1;
            resend_len += len;
            index += len;
        }
        else
        {
            /*超出，返回异常*/
            /*缓冲区已经放不下了*/
            ret = -1;
        }
    }
    return ret;
}
```



其中：

```c
void my_printf(const char *format, ...)
{
    char buffer[256]={0};
    // 使用标准C库函数vsnprintf将可变参数列表格式化为字符串，并存储到buffer中
    va_list args;
    va_start(args, format);
    vsnprintf(buffer, 256, format, args);
    va_end(args);
}
```

是直接调用C语言库的不定参数输出来打印到数组内，虽然这里会花费一点时间，但是目前没找到比这个更好的方式，这里的buffer长度可以根据用户实际情况来设定。

将使用到的变量声明到.h文件中

```c
extern UART_HandleTypeDef *g_uart;
extern DMA_HandleTypeDef *g_dma_tx;
extern int is_resend;
extern int resend_len;
extern char *resend_addr;
```

在中断中进行处理，如果需要再次发送，进行再次发送

```c
void DMA1_Channel4_IRQHandler(void)
{
    /* USER CODE BEGIN DMA1_Channel4_IRQn 0 */
    uint32_t flag_it = g_dma_tx->DmaBaseAddress->ISR;
    uint32_t source_it = g_dma_tx->Instance->CCR;

    /* USER CODE END DMA1_Channel4_IRQn 0 */
    HAL_DMA_IRQHandler(&hdma_usart1_tx);
    /* USER CODE BEGIN DMA1_Channel4_IRQn 1 */

    /* Transfer Complete Interrupt management ***********************************/
    if (((flag_it & (DMA_FLAG_TC1 << g_dma_tx->ChannelIndex)) != RESET) &&
        ((source_it & DMA_IT_TC) != RESET))
    {
        g_uart->gState = HAL_UART_STATE_READY;

        if (is_resend)
        {
            is_resend = 0;
            HAL_UART_Transmit_DMA(g_uart, (uint8_t *)resend_addr, resend_len);
        }
    }

    /* USER CODE END DMA1_Channel4_IRQn 1 */
}
```

~~需要注意的是，这里的`DMA1_Channel4_IRQHandler`是串口发送选择的DMA通过，根据实际使用改变。~~

~~串口DMA发送选择`DMA_NORMAL`模式，在HAL库中是不会执行`void HAL_UART_TxCpltCallback(UART_HandleTypeDef **huart*)`函数的，这个函数是只有`DMA_CIRCULAR`才会执行到；~~

~~同时HAL库的一些清空标志位并没有将串口的标志位设置为Ready，在发送完成之后还是Busy的状态，需要手动清除。~~

~~在需要进行再次发送的情况下，就会把需要发送的数据发送出去。~~

~~在这里判断DMA发送是否完成，根据不同芯片的不同方式不同，但是在HAL库中都可以通过查看系统回调函数`HAL_DMA_IRQHandler`的内容来进行判断，该函数内有着良好的注释，根据需要将系统的方式复制一下就可以了，如F411的芯片为：~~

```c
typedef struct
{
    __IO uint32_t ISR; /*!< DMA interrupt status register */
    __IO uint32_t Reserved0;
    __IO uint32_t IFCR; /*!< DMA interrupt flag clear register */
} DMA_Base_Registers;



uint32_t uart_tx_complete = 0;
uint32_t tmpisr;
/* calculate DMA base and stream number */
DMA_Base_Registers *regs = (DMA_Base_Registers *)g_dma_tx->StreamBaseAddress;

tmpisr = regs->ISR;

if ((tmpisr & (DMA_FLAG_TCIF0_4 << g_dma_tx->StreamIndex)) != RESET)
{
    if (__HAL_DMA_GET_IT_SOURCE(g_dma_tx, DMA_IT_TC) != RESET)
    {
    	uart_tx_complete = 1;
    }
}
```



**更改：**

这里存在一个错误，这里的情况是只有DMA响应发送完成中断，其实串口自己是会有发送完成中断触发的，如下：

```c
void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart)
{
    // __HAL_UART_CLEAR_FLAG(&huart6, UART_FLAG_TC);
    if (huart == g_uart)
    {
        if (is_resend)
        {
            is_resend = 0;
            HAL_UART_Transmit_DMA(g_uart, (uint8_t *)resend_addr, resend_len);
        }
    }
}
```

这样就方便多了，并不需要去查阅dma的回调函数代码，这里的前提是不要改变系统默认的回调函数，如果对于寄存器有详细的了解也可以改写系统的回调函数，否则不建议。

---

其实在这个之前还有一种想法，直接暂停DMA的发送，将数据复制到数组内，将需要发送的数据量增大一些，这个想法是好的，但是实施的时候存在一个问题，在DMA中，数据传输数量寄存器位`CNDTR`,它只能在`DMA_CCRx`的`EN=0`的时候写入，但是在执行的时候`EN=1`，这个时候需要手动关闭，改变这个寄存器的之后再打开，但是又存在一个问题，每当EN打开的时候，它会默认从开始配置的地方运行，这样子会导致buffer的前几个字节会进行多次发送，后面的字节会被丢弃。在这里还有一点，手动更改源寄存器的地址，DMA的外设地址和内存地址分别由`DMA_CPARx`和`DMA_CMARx`来进行控制，但是并不是这样子的，根据查阅数据手册得到：

```
第一个传输的地址是存放在DMA_CPARx / DMA_CMARx寄存器中地址。在传输过程中，这些寄存器保持它们初始的数值，软件不能改变和读出当前正在传输的地址(它在内部的当前外设/存储器地址寄存器中)。
```

所以在这里的`EN=1`开始，应该是芯片内部进行了将`DMA_CPARx / DMA_CMARx`寄存器的地址读取出来，并根据是否需要自增操作来在内部寄存器进行，至此，直接改写buffer失败。



---

同时需要每次发送的数据大致相同，也可以设计为几个buffer，不将buffer拼在一起，这样子在检测到需要发送的时候也能很好的进行，这里的DMA传输缓冲区方案的根本就是存在一个缓冲区，在DMA发送完成中断的之后来判断是否还有需要发送的数据，有的话就把需要发送的数据发送出去。

