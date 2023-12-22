---
title: EC11旋转编码器检测
author: lixinghui
date: 2023-8-30 12:00:00 +0800
categories: [mcu]
tags: [嵌入式杂谈]
---




## 基于定时器的编码器接口

在STM32里面基于定时器的编码器接口来做时非常方便的，同时该编码器接口可以用作电机的旋转检测，这里以HAL库为例：

1、选择定时器的`combined channels`为`encoder mode`，设置为编码器模式之后，相应的通道就会变成高亮显示，如定时器2的通道1和通道2

2、参数设置，设置分频系数为1分频，计数值为1，也就是编码器有位置改变就触发中断，编码器模式选择TI1和TI2，编码器存在干扰，这里可以选择滤波系数为2.

3、开启定时器中断。

4、设置IO口为上拉模式。

5、生成相应的工程文件。



针对生成的工程文件编写相关的代码：

```
__HAL_TIM_CLEAR_IT(&htim2,TIM_IT_UPDATE);
HAL_TIM_Encoder_Start(&htim2, TIM_CHANNEL_ALL);
__HAL_TIM_ENABLE_IT(&htim2,TIM_IT_UPDATE);
```

开启编码器和定时器更新中断。

```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if(htim==&htim2)
    {
        uwDirection=(uint32_t)__HAL_TIM_IS_TIM_COUNTING_DOWN(htim);
        if(uwDirection==1)
        {
            value--;
        }
        else
        {
            value++;
        }
        printf("%d   %d\n",uwDirection,value);
    }
}
```

针对定时器回调函数进行位置判断，其中返回方向0为CW（顺时针旋转），返回方向为1为CCW（逆时针旋转），这里进行增加和减少操作。



到此，基于定时器的编码器接口的EC11旋转编码器检测就完成了，可以说是非常的方便。

参考：

[HAL库EC11](https://blog.csdn.net/Seamfox/article/details/122319487)

[定时器应用-编码器模式](https://blog.csdn.net/weixin_41082463/article/details/105040893)



## 基于普通IO口的位置检测

如果该接口没有定时器的编码器接口，其实是可以手动的采用外部中断或者IO口检测的方式来做的，这里跳过外部中断的方式，直接采用普通IO口来做。

IO口设置为上拉输入模式

这里以三个旋转编码器的检测为例：

```c
#ifndef ENCODER_H
#define ENCODER_H

#include "main.h"

#define H1_R 0X11
#define H1_L 0X12
#define H2_R 0X21
#define H2_L 0X22
#define H3_R 0X31
#define H3_L 0X32

uint8_t GetEncoderValue(void);

#endif // !ENCODER_H
```

```c
#include "encoder.h"

/*macro define encoder IO pin read*/
#define ENCODER_1A_READ() HAL_GPIO_ReadPin(H1A_GPIO_Port, H1A_Pin)
#define ENCODER_1B_READ() HAL_GPIO_ReadPin(H1B_GPIO_Port, H1B_Pin)
#define ENCODER_2A_READ() HAL_GPIO_ReadPin(H2A_GPIO_Port, H2A_Pin)
#define ENCODER_2B_READ() HAL_GPIO_ReadPin(H2B_GPIO_Port, H2B_Pin)
#define ENCODER_3A_READ() HAL_GPIO_ReadPin(H3A_GPIO_Port, H3A_Pin)
#define ENCODER_3B_READ() HAL_GPIO_ReadPin(H3B_GPIO_Port, H3B_Pin)

/*Set the encoder type: 0 is a pulse for a certain bit,1 is a pulse for two positions*/
static uint8_t encoder_type = 0;

typedef struct
{
    uint8_t enc0;
    uint8_t enc1;
    uint8_t enc_old;
    uint8_t encx;
    uint8_t enc_now;
} EncoderParams;

/**
 * @brief calculates the return value of the encoder
 *
 * @param h the pointer of encoder parameters
 * @return uint8_t 0-rotation-free   'R'-Forward turn   'L'-Reverse turn
 */
static uint8_t CalculateEncoderValue(EncoderParams *h)
{
    if (h->enc_now == h->enc_old)
    {
        return (0); /* If the new data is the same as the original (no rotation) */
    }

    if (((h->enc_old == 0x00) && (h->enc_now == 0x02)) ||
        ((h->enc_old == 0x03) && (h->enc_now == 0x01)))
    {
        h->encx = h->enc_now; // 00-10|11-01
    }
    if (((h->enc_old == 0x00) && (h->encx == 0x02) && (h->enc_now == 0x03)) ||
        ((h->enc_old == 0x03) && (h->encx == 0x01) && (h->enc_now == 0x00)))
    {
        /*00-10-11|11-01-00 CW*/
        h->enc_old = h->enc_now, h->encx = 0;
        if (encoder_type == 1) /*two positions, one pulse*/
        {
            return ('R');
        }
        else /*one positions, one pulse*/
        {
            if (h->enc1 == 0)
            {
                h->enc1 = 1;
            }
            else
            {
                h->enc1 = 0;
                return ('R');
            }
        }
    }

    if (((h->enc_old == 0x00) && (h->enc_now == 0x01)) ||
        ((h->enc_old == 0x03) && (h->enc_now == 0x02)))
    {
        h->encx = h->enc_now; /*00-01|11-10*/
    }
    if (((h->enc_old == 0x00) && (h->encx == 0x01) && (h->enc_now == 0x03)) ||
        ((h->enc_old == 0x03) && (h->encx == 0x02) && (h->enc_now == 0x00)))
    {
        /*00-01-11|11-10-00 CCW*/
        h->enc_old = h->enc_now, h->encx = 0;
        if (encoder_type == 1)
        {
            return ('L');
        }
        else
        {
            if (h->enc1 == 0)
            {
                h->enc1 = 1;
            }
            else
            {
                h->enc1 = 0;
                return ('L');
            }
        }
    }
    
    if((h->enc_old==0x01)||(h->enc_old==0x02))
    {
        /*error status,reset the old value*/
        h->enc_old=0x03;
    }
    
    return (0); /*Not decoded correctly*/
}

/*----------------------------------------------------------------*/
/*在主频为98.304MHz时，该函数运行时间在310ticks左右，转换为时间为0.003164ms*/
/*该函数使用GPIO口来检测旋转编码器的转动，需要快速的进入，这里选择1~2ms*/
/*太长时间会导致丢步，无法正常检测到*/
/* When the main frequency is 98.304MHz, the function runs at about 310ticks,
which is converted to a time of 0.003164ms*/
/* This function uses the GPIO port to detect the rotation of the rotary encoder
and requires a fast entry, here choose 1~2ms*/
/* Too long will result in lost steps, and cannot be detected normally */
/*----------------------------------------------------------------*/

/**
 * @brief Scanning of encoder rotation
 *
 * @return uint8_t H1L/H1R/H2L/H2R/H3L/H3R
 */
uint8_t GetEncoderValue(void)
{
    static EncoderParams h1 = {0}, h2 = {0}, h3 = {0};
    uint8_t ret;

    /* H1 encoder detection */
    if (h1.enc0 == 0)
    {
        h1.enc_old = (ENCODER_1A_READ() ? 0x02 : 0x00) + (ENCODER_1B_READ() ? 0x01 : 0x00);
        h1.enc0 = 1; /* Remember the state of the encoder on the first call */
    }
    /*based on the current status of the two IO's Combined into hexadecimal 0x00|0x01|0x02|0x03*/
    h1.enc_now = (ENCODER_1A_READ() ? 0x02 : 0x00) + (ENCODER_1B_READ() ? 0x01 : 0x00);

    /* H2 encoder detection */
    if (h2.enc0 == 0)
    {
        h2.enc_old = (ENCODER_2A_READ() ? 0x02 : 0x00) + (ENCODER_2B_READ() ? 0x01 : 0x00);
        h2.enc0 = 1;
    }
    h2.enc_now = (ENCODER_2A_READ() ? 0x02 : 0x00) + (ENCODER_2B_READ() ? 0x01 : 0x00);

    /* H3 encoder detection */
    if (h3.enc0 == 0)
    {
        h3.enc_old = (ENCODER_3A_READ() ? 0x02 : 0x00) + (ENCODER_3B_READ() ? 0x01 : 0x00);
        h3.enc0 = 1;
    }
    h3.enc_now = (ENCODER_3A_READ() ? 0x02 : 0x00) + (ENCODER_3B_READ() ? 0x01 : 0x00);

    /*H1 encoder calculation */
    ret = CalculateEncoderValue(&h1);
    if (ret == 'R')
        return (H1_R);
    else if (ret == 'L')
        return (H1_L);

    /*H2 encoder calculation */
    ret = CalculateEncoderValue(&h2);
    if (ret == 'R')
        return (H2_R);
    else if (ret == 'L')
        return (H2_L);

    /*H3 encoder calculation */
    ret = CalculateEncoderValue(&h3);
    if (ret == 'R')
        return (H3_R);
    else if (ret == 'L')
        return (H3_L);

    return 0;
}
```



```c
GET_SYSTEM_TICK_STAMP(tick_start);
temp = GetEncoderValue();
GET_SYSTEM_TICK_STAMP(tick_end);

if((temp == H1_R)||(temp == H2_R)||(temp == H3_R))
{
    count++;
    DEBUG_PRINT("%2x,--->>,count=%d\r\n",temp,count);
    #if 0
    printf("GetEncoderValue time:%fms\r\n", CalcSysTickTime(&tick_start, &tick_end));
    printf("GetEncoderValue tick:%lld\r\n", CalcSysTickTicks(&tick_start, &tick_end));
    #endif
}
else if((temp == H1_L)||(temp == H2_L)||(temp == H3_L))
{
    count--;
    DEBUG_PRINT("%2x,<<---,count=%d\r\n",temp,count);
}

HAL_Delay(2);
```

