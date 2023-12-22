---
title: 一种针对没有FPU计算单元的MCU滤波器Q值计算法
author: lixinghui
date: 2023-11-21 14:10:00 +0800
categories: [DSP, filter]
tags: [数字信号处理]
math: true
render_with_liquid: false
---

针对没有浮点计算单元的单片机，在计算滤波器的时候会比较慢，如果能够将浮点数转换为定点数来计算的化，就会快很多。

以一个4阶的低通滤波器为例，分解为二阶滤波器之后，参数为：

```
static float float_b1[] = {0.115258015230152, 0.230516030460303, 0.115258015230152};
static float float_a1[] = {1, -1.11302985416335, 0.574061915083955};
static float float_b2[] = {0.0885793562453461, 0.177158712490692, 0.0885793562453461};
static float float_a2[] = {1, -0.855397932775170, 0.209715357756555};
```

将参数定点化，这里转换为Q20的格式；

```
static q20_t q20_b1[] = {120857, 241714, 120857};
static q20_t q20_a1[] = {1048576, -1167096, 601948};
static q20_t q20_b2[] = {92882, 185764, 92882};
static q20_t q20_a2[] = {1048576, -896950, 219902};
```

在转化完成之后，使用matlab的fvtool工具查看一下转换出来的滤波器和原来的误差。

这里假设“b1,a1”是原始参数，针对Q20的比较为：

```
>> b2=round(b1.*2^20)./2^20;
>> a2=round(a1.*2^20)./2^20;
>> fvtool(b1,a1,b2,a2);
```

> 因为这里为拆分为二阶滤波器的级联来计算的，所以在比较的时候应该还要将二阶滤波器再次级联起来比较一下区别。
{: .prompt-warning }

> 需要实际测量一下，部分精度的丢失在某些时刻会有影响，有些时候又是正常的
{: .prompt-tip }

在这里Q值的选择比较重要，需要根据需要转换的浮点数的精度，同时也需要考虑到原始输入数据的大小，对于32位的单片机，在这里建议使用int32_t的数据类型进行计算，因为超出了32位的大小，单片机计算起来很慢。

这里选择Q20的原因为：输入参数最大为±1024，占11位，这里选择Q20，占21位，加上刚好到32位，同时这里的b值参数相加不会超过1，所以选择Q20，这里的Q值选择需要格外注意。

参考代码：

```c
#include "filter.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef int q31_t;
typedef int q20_t;
typedef short q15_t;
typedef short q14_t;

#define X_SIZE 960
static float float_x[X_SIZE] = {0};
static q31_t q31_x[X_SIZE] = {0};

static float float_b1[] = {0.115258015230152, 0.230516030460303, 0.115258015230152};
static float float_a1[] = {1, -1.11302985416335, 0.574061915083955};
static float float_b2[] = {0.0885793562453461, 0.177158712490692, 0.0885793562453461};
static float float_a2[] = {1, -0.855397932775170, 0.209715357756555};

static q20_t q20_b1[] = {120857, 241714, 120857};
static q20_t q20_a1[] = {1048576, -1167096, 601948};
static q20_t q20_b2[] = {92882, 185764, 92882};
static q20_t q20_a2[] = {1048576, -896950, 219902};

short FloatFilterProcess(short *x, short *y, short len)
{
    short i;
    float sum;
    static float hisx1[3] = {0};
    static float hisy1[3] = {0};
    static float hisx2[3] = {0};
    static float hisy2[3] = {0};
    float *b1 = float_b1;
    float *b2 = float_b2;
    float *a1 = float_a1;
    float *a2 = float_a2;

    if (len > X_SIZE)
    {
        printf("Error: Input buffer too large\r\n");
        return 1;
    }

    for (i = 0; i < len; i++)
    {
        hisx1[0] = x[i];
        sum = b1[0] * hisx1[0] + b1[1] * hisx1[1] + b1[2] * hisx1[2] +
              -a1[1] * hisy1[1] - a1[2] * hisy1[2];
        hisy1[0] = sum;

        hisx1[2] = hisx1[1];
        hisx1[1] = hisx1[0];

        hisy1[2] = hisy1[1];
        hisy1[1] = hisy1[0];
        float_x[i] = hisy1[0];
    }

    for (i = 0; i < len; i++)
    {
        hisx2[0] = float_x[i];
        sum = b2[0] * hisx2[0] + b2[1] * hisx2[1] + b2[2] * hisx2[2] +
              -a2[1] * hisy2[1] - a2[2] * hisy2[2];
        hisy2[0] = sum;

        hisx2[2] = hisx2[1];
        hisx2[1] = hisx2[0];

        hisy2[2] = hisy2[1];
        hisy2[1] = hisy2[0];
        y[i] = hisy2[0] + 0.5f;
    }
    return 0;
}

short QFilterProcess(short *x, short *y, short len)
{
    short i;
    int sum;
    static int hisx1[3] = {0};
    static int hisy1[3] = {0};
    static int hisx2[3] = {0};
    static int hisy2[3] = {0};
    q20_t *b1 = q20_b1;
    q20_t *b2 = q20_b2;
    q20_t *a1 = q20_a1;
    q20_t *a2 = q20_a2;

    if (len > X_SIZE)
    {
        printf("Error: Input buffer too large\r\n");
        return 1;
    }

    for (i = 0; i < len; i++)
    {
        hisx1[0] = x[i];
        sum = b1[0] * hisx1[0] + b1[1] * hisx1[1] + b1[2] * hisx1[2] +
              -a1[1] * hisy1[1] - a1[2] * hisy1[2];
        hisy1[0] = sum >> 20;
        if (sum & 0x80000)/*这里进行四舍五入*/
        {
            hisy1[0] += 1;
        }

        hisx1[2] = hisx1[1];
        hisx1[1] = hisx1[0];
        hisy1[2] = hisy1[1];
        hisy1[1] = hisy1[0];

        q31_x[i] = hisy1[0];
    }

    for (i = 0; i < len; i++)
    {
        hisx2[0] = q31_x[i];
        sum = b2[0] * hisx2[0] + b2[1] * hisx2[1] + b2[2] * hisx2[2] +
              -a2[1] * hisy2[1] - a2[2] * hisy2[2];
        hisy2[0] = sum >> 20;
        if (sum & 0x80000)/*这里进行四舍五入*/
        {
            hisy2[0] += 1;
        }

        hisx2[2] = hisx2[1];
        hisx2[1] = hisx2[0];
        hisy2[2] = hisy2[1];
        hisy2[1] = hisy2[0];

        y[i] = hisy2[0];
    }
    return 0;
}
```

这里的四舍五入操作是判断小数点后一位是否超过0.5，是则加1，否则就舍弃。

这里设置的是Q20，则小数点后一位为第20位，判断是否超过0.5仅需判断该位是否为1即可

第20位为`1<<19`，直接看也就是`0x80000`
