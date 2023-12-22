---
title:  SPI接口HAL库函数转寄存器
author: lixinghui
date: 2023-9-2 12:00:00 +0800
categories: [mcu]
tags: [嵌入式杂谈]
---



直接使用HAL库的函数为：

```c
/**
  * @brief  Transmit and Receive an amount of data in blocking mode.
  * @param  hspi pointer to a SPI_HandleTypeDef structure that contains
  *               the configuration information for SPI module.
  * @param  pTxData pointer to transmission data buffer
  * @param  pRxData pointer to reception data buffer
  * @param  Size amount of data to be sent and received
  * @param  Timeout Timeout duration
  * @retval HAL status
  */
HAL_StatusTypeDef HAL_SPI_TransmitReceive(SPI_HandleTypeDef *hspi, uint8_t *pTxData, uint8_t *pRxData, uint16_t Size,
                                          uint32_t Timeout)
```

如果使用大量的数据一次性发送还好，如果调用这个系统函数只发送1个字节的数据对于通信速度来说就有点慢了。

如果只发送1个字节的数据使用寄存器的方式来发送就会快一些，如下：

```c
uint8_t SPI_WriteByte(SPI_HandleTypeDef *hspi, uint8_t byte)
{
    /* Check if the SPI is already enabled */
    if ((hspi->Instance->CR1 & SPI_CR1_SPE) != SPI_CR1_SPE)
    {
        /* Enable SPI peripheral */
        __HAL_SPI_ENABLE(hspi);
    }

    while (!__HAL_SPI_GET_FLAG(hspi, SPI_FLAG_TXE))
        ;                      /* 等待发送区空 */ 
    hspi->Instance->DR = byte; /* 发送一个byte */ 
    while (!__HAL_SPI_GET_FLAG(hspi, SPI_FLAG_RXNE))
        ;                      /* 等待接收完一个byte */ 
    return hspi->Instance->DR; /* 返回收到的数据 */ 
}
```

这里需要开启SPI的使能，因为HAL库在初始化SPI的时候并没有直接使能SPI接口。

如果这里没有使能会导致数据发送异常，卡死，当然，也可以在初始化的时候进行SPI的使能。