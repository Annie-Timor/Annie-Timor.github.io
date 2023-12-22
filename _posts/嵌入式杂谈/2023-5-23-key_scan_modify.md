---
title: 按键扫描程序修改版
author: lixinghui
date: 2023-5-23 12:00:00 +0800
categories: [mcu]
tags: [嵌入式杂谈]
---


该按键扫描程序支持短按、长按、联合按和连续按下

程序如下：

## `head file`

```c
/*----------------按键扫描文件开始----------------*/

/*----------------包含头文件----------------*/
#include <stdint.h>

/*----------------按键扫描选择----------------*/
#define USE_SHORT_PRESS 0X01
#define USE_LONG_PRESS 0X02
#define USE_UNION_PRESS 0X04
#define USE_CONTINUE_PRESS 0X08
// #define KEY_USE_PRESS (USE_SHORT_PRESS | USE_LONG_PRESS | USE_UNION_PRESS | USE_CONTINUE_PRESS)
#define KEY_USE_PRESS (USE_SHORT_PRESS)

/*----------------按键参数设置----------------*/
#define DEBOUNCE_CNT 5        /*5*10ms=50ms*/
#define LONG_PRESS_CNT 100    /*100*10ms=1000ms*/
#define CONTINUE_PRESS_CNT 20 /*20*10=200ms*/

/*----------------按键返回值设置----------------*/
#define KEY_NONE 0X00
#define KEY1_SHORT_PRESS 0X01
#define KEY2_SHORT_PRESS 0X02
#define KEY3_SHORT_PRESS 0X04
#define KEY1_LONG_PRESS 0X10
#define KEY2_LONG_PRESS 0X20
#define KEY3_LONG_PRESS 0X40
#define KEY12_UNION_PRESS 0X03
#define KEY23_UNION_PRESS 0X06
#define KEY13_UNION_PRESS 0X05
#define KEY1_CONTINUE_PRESS 0X81
#define KEY2_CONTINUE_PRESS 0X82
#define KEY3_CONTINUE_PRESS 0X84

#ifndef TRUE
#define TRUE 1
#endif
#ifndef FALSE
#define FALSE 0
#endif

/*----------------按键端口配置----------------*/
#define KEY1_READ_PIN() HAL_GPIO_ReadPin(KEY0_GPIO_Port, KEY0_Pin)
#define KEY2_READ_PIN() HAL_GPIO_ReadPin(KEY_UP_GPIO_Port, KEY_UP_Pin)

/*----------------按键结构体----------------*/
typedef struct
{
    uint8_t current_state;  /*按键当前状态*/
    uint8_t previous_state; /*按键上一状态*/
    uint8_t release_state;  /*1表示释放，0表示未释放，ff为联合按键钳位*/
    uint16_t press_count;   /*按键时长计数*/
    uint8_t return_value;   /*按键返回状态*/
} FSM_KeyHandle;

uint8_t KeyScan(void);

/*----------------按键扫描文件结束----------------*/
```



## `source file`

```c

static FSM_KeyHandle key = {0};

/**
 @brief 按键检测，建议每10ms检测一次
 @param none
 @return 按键的状态
*/
uint8_t KeyScan(void)
{
    key.current_state = 0;
    key.return_value = KEY_NONE;

    /*读取按键IO口状态*/
    if (KEY1_READ_PIN() == GPIO_PIN_SET)
    {
        key.current_state |= KEY1_SHORT_PRESS;
    }
    if (KEY2_READ_PIN() == GPIO_PIN_SET)
    {
        key.current_state |= KEY2_SHORT_PRESS;
    }

    if (key.current_state == 0)
    {
        /*按键被释放*/
        if (key.previous_state != 0) /*按键刚被释放*/
        {
            /*消抖*/
            if ((key.press_count >= DEBOUNCE_CNT) && (key.release_state != 0xff))
            {
                if ((key.press_count <= LONG_PRESS_CNT)||
                    (KEY_USE_PRESS==USE_SHORT_PRESS))
                {
                    if (key.previous_state == KEY1_SHORT_PRESS)
                    {
                        key.return_value = KEY1_SHORT_PRESS;
                    }
                    else if (key.previous_state == KEY2_SHORT_PRESS)
                    {
                        key.return_value = KEY2_SHORT_PRESS;
                    }
                    else if (key.previous_state == KEY3_SHORT_PRESS)
                    {
                        key.return_value = KEY3_SHORT_PRESS;
                    }
                }
            }
            /*清空上一个状态和按键计数*/
            key.previous_state = KEY_NONE;
            key.press_count = 0;
        }
        /*标记按键释放*/
        key.release_state = TRUE;
    }
    else
    {
        /*按键正在被按下*/
        if (key.release_state == TRUE)
        {
            key.release_state = FALSE;
        }

        /*当前值和上一个值不一致----当前状态刚按下*/
        if (key.previous_state != key.current_state)
        {
            key.previous_state = key.current_state;
            key.press_count = 0;
        }
        else /*持续按下*/
        {
            key.press_count++;

            if (key.press_count >= DEBOUNCE_CNT) /*消抖*/
            {
                if (key.release_state == FALSE)
                {

#if (KEY_USE_PRESS & USE_LONG_PRESS)
                    if ((key.press_count >= LONG_PRESS_CNT)&&
                        (key.press_count < (LONG_PRESS_CNT+5)))
                    {
                        key.press_count+=5;
                        if (key.current_state == KEY1_SHORT_PRESS)
                        {
                            key.return_value = KEY1_LONG_PRESS;
                        }
                        else if (key.current_state == KEY2_SHORT_PRESS)
                        {
                            key.return_value = KEY2_LONG_PRESS;
                        }
                        else if (key.current_state == KEY3_SHORT_PRESS)
                        {
                            key.return_value = KEY3_LONG_PRESS;
                        }
                    }
#endif

#if (KEY_USE_PRESS & USE_CONTINUE_PRESS)
                    if ((key.press_count >= LONG_PRESS_CNT) && 
                        (key.press_count % CONTINUE_PRESS_CNT == 0))
                    {
                        if (key.current_state == KEY1_SHORT_PRESS)
                        {
                            key.return_value = KEY1_CONTINUE_PRESS;
                        }
                        else if (key.current_state == KEY2_SHORT_PRESS)
                        {
                            key.return_value = KEY2_CONTINUE_PRESS;
                        }
                        else if (key.current_state == KEY3_SHORT_PRESS)
                        {
                            key.return_value = KEY3_CONTINUE_PRESS;
                        }
                    }
#endif

#if (KEY_USE_PRESS & USE_UNION_PRESS)
                    if (key.current_state == KEY12_UNION_PRESS)
                    {
                        key.return_value = KEY12_UNION_PRESS;
                        key.release_state = 0xff;
                    }
                    else if (key.current_state == KEY13_UNION_PRESS)
                    {
                        key.return_value = KEY13_UNION_PRESS;
                        key.release_state = 0xff;
                    }
                    else if (key.current_state == KEY23_UNION_PRESS)
                    {
                        key.return_value = KEY23_UNION_PRESS;
                        key.release_state = 0xff;
                    }
#endif
                }
            }
        }
    }

    return key.return_value;
}

```



## `example`

```c
		uint8_t key_state = KeyScan();
        if(key_state!=KEY_NONE)
        {
            printf("key_state:0x%02x\r\n",key_state);
        }
        HAL_Delay(10);
```

