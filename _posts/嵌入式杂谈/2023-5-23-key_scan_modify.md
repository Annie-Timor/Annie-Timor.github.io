---
title: 按键扫描程序修改版
author: lixinghui
date: 2023-5-23 12:00:00 +0800
categories: [mcu]
tags: [嵌入式杂谈]
---


该按键扫描程序支持短按、长按、双击、和连续按下，删除了联合按下

程序如下：

## `head file`

```c
/*----------------按键扫描文件开始----------------*/

/*----------------包含头文件----------------*/
#include <stdint.h>

/*----------------按键扫描选择----------------*/
#define USE_SHORT_PRESS 		0X01
#define USE_DOUBLE_PRESS 		0X10
#define USE_LONG_PRESS 			0X02
#define USE_UNION_PRESS 		0X04
#define USE_CONTINUE_PRESS 		0X08

/*----------------按键状态值设置----------------*/
#define KEY_NONE 				0X00
#define KEY_PRESS 		        0X01
#define KEY_RELEASE 		    0X02
#define KEY_SHORT_PRESS 	    0X0A
#define KEY_DOUBLE_PRESS 	    0X0B
#define KEY_LONG_PRESS 	        0X0C
#define KEY_CONTINUE_PRESS 	    0X0D

/*----------------按键返回值设置----------------*/
#define KEY_NONE                0X00
#define KEY1_SHORT_PRESS        0X01
#define KEY2_SHORT_PRESS        0X02
#define KEY3_SHORT_PRESS        0X04
#define KEY1_DOUBLE_PRESS       0X81
#define KEY2_DOUBLE_PRESS       0X82
#define KEY3_DOUBLE_PRESS       0X84
#define KEY1_RELEASE            0X11
#define KEY2_RELEASE            0X12
#define KEY3_RELEASE            0X14
#define KEY1_LONG_PRESS         0X21
#define KEY2_LONG_PRESS         0X22
#define KEY3_LONG_PRESS         0X24
#define KEY1_CONTINUE_PRESS     0X31
#define KEY2_CONTINUE_PRESS     0X32
#define KEY3_CONTINUE_PRESS     0X34



/*----------------按键端口配置----------------*/
#define KEY1_READ_PIN() HAL_GPIO_ReadPin(KEY0_GPIO_Port, KEY0_Pin)
#define KEY2_READ_PIN() HAL_GPIO_ReadPin(KEY_UP_GPIO_Port, KEY_UP_Pin)

/*----------------按键结构体----------------*/
typedef struct
{
    uint8_t detect_mode;    /*检测模式*/
    uint8_t current_state;  /*按键当前状态*/
    uint8_t previous_state; /*按键上一状态*/
    uint8_t release_state;  /*1表示释放，0表示未释放，ff为联合按键钳位*/
    uint16_t press_count;   /*按下时长计数*/
    uint16_t release_count; /*释放时长计数*/
    uint8_t press_no_ret;   /*按下按键无返回*/
    uint8_t return_value;   /*按键返回状态*/
} KeyHandle;

void KeyInit(void);
uint8_t KeyScan(void);

/*----------------按键扫描文件结束----------------*/
```



## `source file`

```c
/*----------------按键参数设置----------------*/
#define DEBOUNCE_CNT 5        /*5*10ms=50ms*/
#define LONG_PRESS_CNT 100    /*100*10ms=1000ms*/
#define CONTINUE_PRESS_CNT 20 /*20*10=200ms*/

#ifndef TRUE
#define TRUE 1
#endif
#ifndef FALSE
#define FALSE 0
#endif

static KeyHandle key1 = {0};
static KeyHandle key2 = {0};
static KeyHandle key3 = {0};

/**
 * @brief 按键初始化
 * 配置单独的按键支持哪些按下模式
 */
void KeyInit(void)
{
    key1.detect_mode = USE_SHORT_PRESS | USE_DOUBLE_PRESS | USE_LONG_PRESS;
    key2.detect_mode = USE_SHORT_PRESS | USE_CONTINUE_PRESS | USE_LONG_PRESS;
}

/**
 * @brief 按键检测
 * 
 * @param key 单独按键参数结构体
 * @return uint8_t 按键值
 */
uint8_t KeyJudgement(KeyHandle *key)
{
    key->return_value = KEY_NONE;
    if (key->current_state == KEY_NONE) /*按键被释放*/
    {
        key->release_count++;
        /*消抖*/
        if (key->press_count >= DEBOUNCE_CNT)
        {
            /*短按/小于长按时间或者不支持长按*/
            if ((key->press_count < LONG_PRESS_CNT) ||
                !(key->detect_mode & USE_LONG_PRESS))
            {
                if (key->detect_mode & USE_DOUBLE_PRESS)
                { /*支持双击模式*/
                    if (key->press_no_ret)
                    {
                        /*返回双击*/
                        key->press_no_ret = 0x00;
                        key->return_value = KEY_DOUBLE_PRESS;
                    }
                    else
                    {
                        /*单击暂存*/
                        key->press_no_ret |= 0x01;
                    }
                }
                else /*不支持双击模式，直接返回*/
                {
                    key->return_value = KEY_SHORT_PRESS;
                }
            }
            else
            {
                /*长按*/
                key->return_value = KEY_LONG_PRESS;
            }
        }

        if ((key->press_no_ret) && (key->release_count > 50))
        {
            key->press_no_ret = 0;
            key->return_value = KEY_SHORT_PRESS;
        }

        if (key->previous_state != KEY_NONE) /*按键刚被释放*/
        {
            key->release_count = 0;
            /*清空上一个状态和按键计数*/
            key->previous_state = KEY_NONE;
            key->press_count = 0;
            // key->return_value = KEY_RELEASE;
        }
    }
    else
    {
        /*当前值和上一个值不一致----当前状态刚按下*/
        if (key->previous_state != key->current_state)
        {
            key->previous_state = key->current_state;
            key->press_count = 0;
            key->release_count = 0;
        }
        else /*持续按下*/
        {
            key->press_count++;

            if (key->press_count >= DEBOUNCE_CNT) /*消抖*/
            {
                if (key->detect_mode & USE_CONTINUE_PRESS)
                { /*支持连续模式*/
                    if ((key->press_count >= LONG_PRESS_CNT) &&
                        (key->press_count % CONTINUE_PRESS_CNT == 0))
                    {
                        // if (key->current_state == KEY_PRESS)
                        {
                            key->return_value = KEY_CONTINUE_PRESS;
                        }
                    }
                } // USE_CONTINUE_PRESS
            }
        }
    }

    return key->return_value;
}

/**
 @brief 按键检测，建议每10ms检测一次
 @param none
 @return 按键的状态
*/
uint8_t KeyScan(void)
{
    uint8_t value = KEY_NONE;

    /*这里可以根据需求进行联合按键的检测*/
    /*目前针对多个按键同一瞬间按下/释放的情况，会丢失数据，可以使用每个按键一个返回值来做*/
    /*用户根据需要修改这个函数*/

    key1.current_state = KEY_NONE;
    key2.current_state = KEY_NONE;
    /*读取按键IO口状态*/
    if (KEY1_READ_PIN() == GPIO_PIN_SET)
    {
        key1.current_state |= KEY_PRESS;
    }
    if (KEY2_READ_PIN() == GPIO_PIN_SET)
    {
        key2.current_state |= KEY_PRESS;
    }

    KeyJudgement(&key1);
    KeyJudgement(&key2);

    if (key1.return_value == KEY_SHORT_PRESS)
    {
        value = KEY1_SHORT_PRESS;
    }
    else if (key1.return_value == KEY_DOUBLE_PRESS)
    {
        value = KEY1_DOUBLE_PRESS;
    }
    else if (key1.return_value == KEY_LONG_PRESS)
    {
        value = KEY1_LONG_PRESS;
    }
    else if (key1.return_value == KEY_CONTINUE_PRESS)
    {
        value = KEY1_CONTINUE_PRESS;
    }
    else if (key1.return_value == KEY_RELEASE)
    {
        value = KEY1_RELEASE;
    }

    if (key2.return_value == KEY_SHORT_PRESS)
    {
        value = KEY2_SHORT_PRESS;
    }
    else if (key2.return_value == KEY_DOUBLE_PRESS)
    {
        value = KEY2_DOUBLE_PRESS;
    }
    else if (key2.return_value == KEY_LONG_PRESS)
    {
        value = KEY2_LONG_PRESS;
    }
    else if (key2.return_value == KEY_CONTINUE_PRESS)
    {
        value = KEY2_CONTINUE_PRESS;
    }
    else if (key2.return_value == KEY_RELEASE)
    {
        value = KEY2_RELEASE;
    }

    return value;
}

```



## `example`

```c
		HAL_Delay(10);
        
        key_status = KeyScan2();
        switch (key_status)
        {
        case KEY1_SHORT_PRESS:
            printf("KEY1_SHORT_PRESS\n");
            break;
        case KEY2_SHORT_PRESS:
            printf("KEY2_SHORT_PRESS\n");
            break;
        case KEY1_DOUBLE_PRESS:
            printf("KEY1_DOUBLE_PRESS\n");
            break;
        case KEY2_DOUBLE_PRESS:
            printf("KEY2_DOUBLE_PRESS\n");
            break;
        case KEY1_RELEASE:
            printf("KEY1_RELEASE\n");
            break;
        case KEY2_RELEASE:
            printf("KEY2_RELEASE\n");
            break;
        case KEY1_LONG_PRESS:
            printf("KEY1_LONG_PRESS\n");
            break;
        case KEY2_LONG_PRESS:
            printf("KEY2_LONG_PRESS\n");
            break;
        case KEY1_CONTINUE_PRESS:
            printf("KEY1_CONTINUE_PRESS\n");
            break;
        case KEY2_CONTINUE_PRESS:
            printf("KEY2_CONTINUE_PRESS\n");
            break;
        case KEY12_UNION_PRESS:
            printf("KEY1_2_UNION_PRESS\n");
            break;
        default:
            break;
        }
```

