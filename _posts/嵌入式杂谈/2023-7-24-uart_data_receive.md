---
title: 串口数据接收
author: lixinghui
date: 2023-7-24 12:00:00 +0800
categories: [mcu]
tags: [嵌入式杂谈]
---



## 空闲中断+DMA

针对不定长数据接收，在STM32芯片中最常见的方式就是使用串口空闲中断+DMA的方式来接收数据，这种在有着硬件支持的情况下是最方便的，数据通过DMA的方式搬运到指定的数组内，在检测到接收空闲的时候触发中断，这时候一包数据就已经传输完成了，只需要读取指定数据内的数据即可。

生成的伪代码如下：

```c
/********** uart.h **********/

#ifndef TRUE
#define TRUE 1
#endif
#ifndef FALSE
#define FALSE 0
#endif

#define MAX_RECEIVE_LEN 64

/* UART receive data define */
typedef struct
{
    uint8_t data[MAX_RECEIVE_LEN];
    uint8_t length;
    uint8_t trigger;
}UartReceiveHandle;

/********** main.c **********/
/* start uart1 idle interrupt */
__HAL_UART_ENABLE_IT(&huart1, UART_IT_IDLE);
/* start the uart1 dma receive */
HAL_UART_Receive_DMA(&huart1, (uint8_t *)uart_recv.data, MAX_RECEIVE_LEN);


/********** stm32f1xx_it.c **********/
void USART1_IRQHandler(void)
{
    /* USER CODE BEGIN USART1_IRQn 0 */

    /* USER CODE END USART1_IRQn 0 */
    //   HAL_UART_IRQHandler(&huart1);
    /* USER CODE BEGIN USART1_IRQn 1 */
    /* disable the default IRQHandler */
    USER_UART_IRQHandler(&huart1);
    /* USER CODE END USART1_IRQn 1 */
}

void USER_UART_IRQHandler(UART_HandleTypeDef *huart)
{
    if (USART1 == huart->Instance)
    {   /* Check whether the interrupt is idle */
        if (RESET != __HAL_UART_GET_FLAG(&huart1, UART_FLAG_IDLE))
        {
            /* Clear idle interrupt flag (otherwise it will keep entering interrupt) */
            __HAL_UART_CLEAR_IDLEFLAG(&huart1);

            /* Stop this DMA transfer */
            HAL_UART_DMAStop(&huart1);
            /* Calculate the length of the received data */
            uart_recv.length = MAX_RECEIVE_LEN - __HAL_DMA_GET_COUNTER(&hdma_usart1_rx);
            uart_recv.trigger = TRUE;
        }
    }
}

/********** receive data process ***********/
if (uart_recv.trigger==TRUE)
{
    /* user process code */
    HAL_UART_Transmit(&huart1,uart_recv.data,uart_recv.length,0xff);
    /* user process end*/

    /* clear receive data */
    uart_recv.trigger=FALSE;
    uart_recv.length=0;
    memset(uart_recv.data,0,MAX_RECEIVE_LEN);
    /* restart the uart1 dma receive */
    HAL_UART_Receive_DMA(&huart1, (uint8_t *)uart_recv.data, MAX_RECEIVE_LEN);
}
```

## 空闲中断

在有空闲中断的时候，仅需要接收每个字节然后手动搬运到数组内就可以了，在检测到空闲中断的时候就表示这一包数据接收完成了。

生成的伪代码如下：

```c
/* start uart1 idle&rxne interrupt */
__HAL_UART_ENABLE_IT(&huart1, UART_IT_RXNE);
__HAL_UART_ENABLE_IT(&huart1, UART_IT_IDLE);


void USER_UART_IRQHandler(UART_HandleTypeDef *huart)
{
    uint8_t receive_byte=0;
    if (USART1 == huart->Instance)
    {   
        /* Check whether the interrupt is Receive Data register not empty */ 
        if (RESET != __HAL_UART_GET_FLAG(&huart1, UART_FLAG_RXNE))
        {
            HAL_UART_Receive(&huart1, &receive_byte, 1, 0xff);
            uart_recv.data[uart_recv.length++]=receive_byte;
            if(uart_recv.length==MAX_RECEIVE_LEN)
            {   /* prevent receive data oversize the buffer */
                uart_recv.length=MAX_RECEIVE_LEN-1;
            }

            __HAL_UART_CLEAR_FLAG(&huart1, UART_FLAG_RXNE);
        }
        /* Check whether the interrupt is idle */
        if (RESET != __HAL_UART_GET_FLAG(&huart1, UART_FLAG_IDLE))
        {
            /* Clear idle interrupt flag (otherwise it will keep entering interrupt) */
            __HAL_UART_CLEAR_IDLEFLAG(&huart1);
            uart_recv.trigger = TRUE;
        }
    }
}

/* receive data process */
if (uart_recv.trigger==TRUE)
{
    /* user process code */
    HAL_UART_Transmit(&huart1,uart_recv.data,uart_recv.length,0xff);
    /* user process end*/

    /* clear receive data */
    uart_recv.trigger=FALSE;
    uart_recv.length=0;
    memset(uart_recv.data,0,MAX_RECEIVE_LEN);
}
```

## 单数据接收中断

在只有中断数据接收中断的时候，通过手动将接收到的数据传入目标数组，同时手动判断是否接收完成了，进入伪空闲中断模式。

这里的判断是否接收完成需要一个时间基准，在每次接收到数据的时候更新这个值，通过1ms的系统定时器中断来判断是否接收完成了。这里需要注意，不同波特率的情况下判断数据是否接收完成的条件不同，如：

9600bps=960bytes per second，这时接收一个字节差不多就需要1ms，就需要加大中断的判断时间。

115200bps=11520bytes per second，这时候接收一个字节仅需要0.1ms左右，在1ms内都没有接收到新的数据就可以判断该包数据已经传输完成了。

伪代码如下：

```c
/* UART receive data define */
typedef struct
{
    uint8_t data[MAX_RECEIVE_LEN];
    uint8_t length;
    uint8_t length_mark;
    uint8_t trigger;
    uint32_t ms;
    uint32_t tick;
}UartReceiveHandle;

/* start uart1 rxne interrupt */
__HAL_UART_ENABLE_IT(&huart1, UART_IT_RXNE);

void USER_UART_IRQHandler(UART_HandleTypeDef *huart)
{
    uint8_t receive_byte = 0;
    if (USART1 == huart->Instance)
    {
        /* Check whether the interrupt is Receive Data register not empty */
        if (RESET != __HAL_UART_GET_FLAG(&huart1, UART_FLAG_RXNE))
        {
            HAL_UART_Receive(&huart1, &receive_byte, 1, 0xff);
            uart_recv.data[uart_recv.length++] = receive_byte;
            if (uart_recv.length == MAX_RECEIVE_LEN)
            { /* prevent receive data oversize the buffer */
                uart_recv.length = MAX_RECEIVE_LEN - 1;
            }

            __HAL_UART_CLEAR_FLAG(&huart1, UART_FLAG_RXNE);

            uart_recv.ms = HAL_GetTick();
            uart_recv.tick = SysTick->VAL;
        }
    }
}

void SysTick_Handler(void)
{
    /* USER CODE BEGIN SysTick_IRQn 0 */

    /* USER CODE END SysTick_IRQn 0 */
    HAL_IncTick();
    /* USER CODE BEGIN SysTick_IRQn 1 */
    if ((uart_recv.length == uart_recv.length_mark) &&
        (uart_recv.ms != HAL_GetTick()) && 
        (uart_recv.length!=0))
    {
        uart_recv.trigger = 1;
    }
    else
    {
        uart_recv.length_mark=uart_recv.length;
    }
    /* USER CODE END SysTick_IRQn 1 */
}
```

当然，以上的中断接收仅以ST的芯片作为样例，其他芯片只要采用同样的实现方式一样可行。