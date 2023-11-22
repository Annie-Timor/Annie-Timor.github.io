---
title: 基于SysTick的us级延时
author: lixinghui
date: 2023-6-15 12:00:00 +0800
categories: [mcu]
tags: [mcu]
---


在默认的HAL程序中，是只有ms级的硬延时函数，没有us级延迟函数的，但是它的ms级延时函数是基于SysTick定时器来运行的，根据这一点我们也可以基于SysTick做一个us级别的延时函数。

## SysTick寄存器

通过对SysTick控制与状态寄存器(CTRL)的设置，可选择AHB时钟(HCLK)8分频或Cortex(HCLK)时钟作为SysTick时钟。

重装载值寄存器(LOAD)，范围为`0x00000001-0x00FFFFFF`,24bit的定时器，

当前值寄存器(VAL)，记录当前寄存器的值，递减，24bit

校准值寄存器(CALIB)

## SysTick初始化

```c
/**
  \brief   System Tick Configuration
  \details Initializes the System Timer and its interrupt, and starts the System Tick Timer.
           Counter is in free running mode to generate periodic interrupts.
  \param [in]  ticks  Number of ticks between two interrupts.
  \return          0  Function succeeded.
  \return          1  Function failed.
  \note    When the variable <b>__Vendor_SysTickConfig</b> is set to 1, then the
           function <b>SysTick_Config</b> is not included. In this case, the file <b><i>device</i>.h</b>
           must contain a vendor-specific implementation of this function.
 */
__STATIC_INLINE uint32_t SysTick_Config(uint32_t ticks)
{
  if ((ticks - 1UL) > SysTick_LOAD_RELOAD_Msk)
  {
    return (1UL);                                                   /* Reload value impossible */
  }

  SysTick->LOAD  = (uint32_t)(ticks - 1UL);                         /* set reload register */
  NVIC_SetPriority (SysTick_IRQn, (1UL << __NVIC_PRIO_BITS) - 1UL); /* set Priority for Systick Interrupt */
  SysTick->VAL   = 0UL;                                             /* Load the SysTick Counter Value */
  SysTick->CTRL  = SysTick_CTRL_CLKSOURCE_Msk |
                   SysTick_CTRL_TICKINT_Msk   |
                   SysTick_CTRL_ENABLE_Msk;                         /* Enable SysTick IRQ and SysTick Timer */
  return (0UL);                                                     /* Function successful */
}

```



## us级别延时

```c
/**
 * @brief use systick timer do while loop to delay us time
 *
 * @param nus the user delay n us
 */
void UserDelayUs(uint32_t nus)
{
    /*the hal function already set systick callback is 1ms*/
    uint32_t us_cnt = (SysTick->LOAD + 1) / 1000; /*get per us need tick cnt*/
    int32_t total_cnt = us_cnt * nus;
    uint32_t tick_prev = SysTick->VAL; /*get systick now value*/
    uint32_t tick_curr = SysTick->VAL;
    uint16_t tick_diff = 0;

    while (total_cnt > 0)
    {
        tick_curr = SysTick->VAL;
        if (tick_prev > tick_curr)
        {
            tick_diff = tick_prev - tick_curr;
        }
        else
        {
            /*the systick execute once reload*/
            tick_diff = tick_prev + (SysTick->LOAD + 1) - tick_curr;
        }
        total_cnt -= tick_diff;
        tick_prev = tick_curr;
    }
}
```

## 运行时间测量

```c
typedef struct
{
    uint32_t msec;
    uint32_t count;
    uint32_t load;
} SysTickParams;


#define GET_SYSTEM_TICK_STAMP(params)    \
{                                        \
    params.msec = HAL_GetTick();         \
    params.load = (SysTick->LOAD + 1);   \
    params.count = SysTick->VAL;         \
}

/**
 * @brief Get the Sys Tick Stamp object
 * 
 * @param params struct point
 */
void GetSysTickStamp(SysTickParams *params)
{
    params->msec = HAL_GetTick();
    params->load = (SysTick->LOAD + 1);
    params->count = SysTick->VAL;
}

/**
 * @brief according to the start and end timestamp,
 * calculate cost time 
 * @param start start time struct point
 * @param end end time struct point
 * @return float ms time
 */
float CalcSysTickTime(SysTickParams *start, SysTickParams *end)
{
    float ms, us, ret;
    if (end->count > start->count)
    {
        us = start->load + start->count - end->count;
        us = us * 1000 / end->load;
        ms = end->msec - 1 - start->msec;
    }
    else
    {
        us = start->count - end->count;
        us = us * 1000 / end->load;
        ms = end->msec - start->msec;
    }
    ret = ms + us / 1000.0f;
    return ret;
}

#if 1
/**
 * @brief according to the start and end timestamp,
 * calculate cost time,ticks
 * @param start start time struct point
 * @param end end time struct point
 * @return long long mcu ticks
 */
long long CalcSysTickTicks(SysTickParams *start, SysTickParams *end)
{
    long long ticks;

    if (end->count > start->count)
    {
        ticks=start->load + start->count - end->count;
        ticks += (end->msec - 1 - start->msec)*end->load;
    }
    else
    {
        ticks=start->count - end->count;
        ticks += (end->msec - start->msec)*end->load;
    }
    return ticks;
}
#endif
```

example:

```c
		GetSysTickStamp(&tick1);
#if 0
        /*calculate*/
        in = 1.0f;
        for (i = 0; i < 100; i++)
        {
            out = sqrt(in);
            in = in + 1;
        }
        /*end*/
#endif
        UserDelayUs(1000);

        GetSysTickStamp(&tick2);

        /*cout the result*/
        printf("time:%fms\r\n", CalcSysTickTime(&tick1, &tick2));
```

输出结果：

```
time:1.001264ms
```

中间不做任何处理：

```
time:0.000681ms
ticks:49
```

改为宏定义获取时间：

```
time:0.000361ms
ticks:26
```



延时测试（主频72MHz）：

```
delay 100 us, calcTime 0.101069 ms, calcTicks 7277
delay 200 us, calcTime 0.201181 ms, calcTicks 14485
delay 300 us, calcTime 0.301181 ms, calcTicks 21685
delay 400 us, calcTime 0.401181 ms, calcTicks 28885
delay 500 us, calcTime 0.501139 ms, calcTicks 36082
delay 600 us, calcTime 0.601139 ms, calcTicks 43282
delay 700 us, calcTime 0.701181 ms, calcTicks 50485
delay 800 us, calcTime 0.801139 ms, calcTicks 57682
delay 900 us, calcTime 0.901139 ms, calcTicks 64882
delay 1000 us, calcTime 1.001139 ms, calcTicks 72082
delay 1100 us, calcTime 1.101139 ms, calcTicks 79282
delay 1200 us, calcTime 1.201125 ms, calcTicks 86481
delay 1300 us, calcTime 1.301125 ms, calcTicks 93681
delay 1400 us, calcTime 1.401139 ms, calcTicks 100882
delay 1500 us, calcTime 1.501070 ms, calcTicks 108077
delay 1600 us, calcTime 1.601125 ms, calcTicks 115281
delay 1700 us, calcTime 1.701167 ms, calcTicks 122484
delay 1800 us, calcTime 1.801181 ms, calcTicks 129685
delay 1900 us, calcTime 1.901125 ms, calcTicks 136881
delay 2000 us, calcTime 2.001083 ms, calcTicks 144078
delay 2100 us, calcTime 2.101153 ms, calcTicks 151283
delay 2200 us, calcTime 2.201111 ms, calcTicks 158480
delay 2300 us, calcTime 2.301181 ms, calcTicks 165685
delay 2400 us, calcTime 2.401111 ms, calcTicks 172880
delay 2500 us, calcTime 2.501194 ms, calcTicks 180086
delay 2600 us, calcTime 2.600986 ms, calcTicks 187271
delay 2700 us, calcTime 2.701083 ms, calcTicks 194478
delay 2800 us, calcTime 2.801153 ms, calcTicks 201683
delay 2900 us, calcTime 2.901028 ms, calcTicks 208874
delay 3000 us, calcTime 3.001083 ms, calcTicks 216078
delay 3100 us, calcTime 3.101069 ms, calcTicks 223277
delay 3200 us, calcTime 3.201083 ms, calcTicks 230478
delay 3300 us, calcTime 3.301083 ms, calcTicks 237678
delay 3400 us, calcTime 3.401097 ms, calcTicks 244879
delay 3500 us, calcTime 3.501181 ms, calcTicks 252085
delay 3600 us, calcTime 3.601181 ms, calcTicks 259285
delay 3700 us, calcTime 3.701083 ms, calcTicks 266478
delay 3800 us, calcTime 3.801139 ms, calcTicks 273682
delay 3900 us, calcTime 3.901056 ms, calcTicks 280876
delay 4000 us, calcTime 4.001056 ms, calcTicks 288076
delay 4100 us, calcTime 4.101083 ms, calcTicks 295278
delay 4200 us, calcTime 4.201069 ms, calcTicks 302477
delay 4300 us, calcTime 4.301055 ms, calcTicks 309676
delay 4400 us, calcTime 4.401083 ms, calcTicks 316878
delay 4500 us, calcTime 4.501181 ms, calcTicks 324085
delay 4600 us, calcTime 4.601056 ms, calcTicks 331276
delay 4700 us, calcTime 4.701014 ms, calcTicks 338473
delay 4800 us, calcTime 4.801000 ms, calcTicks 345672
delay 4900 us, calcTime 4.901000 ms, calcTicks 352872
delay 5000 us, calcTime 5.001111 ms, calcTicks 360080
delay 5100 us, calcTime 5.101069 ms, calcTicks 367277
delay 5200 us, calcTime 5.201042 ms, calcTicks 374475
delay 5300 us, calcTime 5.301042 ms, calcTicks 381675
delay 5400 us, calcTime 5.401181 ms, calcTicks 388885
delay 5500 us, calcTime 5.501181 ms, calcTicks 396085
delay 5600 us, calcTime 5.601111 ms, calcTicks 403280
delay 5700 us, calcTime 5.701139 ms, calcTicks 410482
delay 5800 us, calcTime 5.801097 ms, calcTicks 417679
delay 5900 us, calcTime 5.901139 ms, calcTicks 424882
delay 6000 us, calcTime 6.001000 ms, calcTicks 432072
delay 6100 us, calcTime 6.101153 ms, calcTicks 439283
delay 6200 us, calcTime 6.201083 ms, calcTicks 446478
delay 6300 us, calcTime 6.301111 ms, calcTicks 453680
delay 6400 us, calcTime 6.401000 ms, calcTicks 460872
delay 6500 us, calcTime 6.501139 ms, calcTicks 468082
delay 6600 us, calcTime 6.601042 ms, calcTicks 475275
delay 6700 us, calcTime 6.701194 ms, calcTicks 482486
delay 6800 us, calcTime 6.801153 ms, calcTicks 489683
delay 6900 us, calcTime 6.901181 ms, calcTicks 496885
delay 7000 us, calcTime 7.001181 ms, calcTicks 504085
delay 7100 us, calcTime 7.101194 ms, calcTicks 511286
delay 7200 us, calcTime 7.201069 ms, calcTicks 518477
delay 7300 us, calcTime 7.301097 ms, calcTicks 525679
delay 7400 us, calcTime 7.401139 ms, calcTicks 532882
delay 7500 us, calcTime 7.501139 ms, calcTicks 540082
delay 7600 us, calcTime 7.601000 ms, calcTicks 547272
delay 7700 us, calcTime 7.701180 ms, calcTicks 554485
delay 7800 us, calcTime 7.801000 ms, calcTicks 561672
delay 7900 us, calcTime 7.901167 ms, calcTicks 568884
delay 8000 us, calcTime 8.001014 ms, calcTicks 576073
delay 8100 us, calcTime 8.101042 ms, calcTicks 583275
delay 8200 us, calcTime 8.201138 ms, calcTicks 590482
delay 8300 us, calcTime 8.301014 ms, calcTicks 597673
delay 8400 us, calcTime 8.401055 ms, calcTicks 604876
delay 8500 us, calcTime 8.500986 ms, calcTicks 612071
delay 8600 us, calcTime 8.601181 ms, calcTicks 619285
delay 8700 us, calcTime 8.701097 ms, calcTicks 626479
delay 8800 us, calcTime 8.801139 ms, calcTicks 633682
delay 8900 us, calcTime 8.901139 ms, calcTicks 640882
delay 9000 us, calcTime 9.001041 ms, calcTicks 648075
delay 9100 us, calcTime 9.101111 ms, calcTicks 655280
delay 9200 us, calcTime 9.201111 ms, calcTicks 662480
delay 9300 us, calcTime 9.300986 ms, calcTicks 669671
delay 9400 us, calcTime 9.400986 ms, calcTicks 676871
delay 9500 us, calcTime 9.501056 ms, calcTicks 684076
delay 9600 us, calcTime 9.600986 ms, calcTicks 691271
delay 9700 us, calcTime 9.701180 ms, calcTicks 698485
delay 9800 us, calcTime 9.801181 ms, calcTicks 705685
delay 9900 us, calcTime 9.901180 ms, calcTicks 712885
delay 10000 us, calcTime 10.001194 ms, calcTicks 720086
```

