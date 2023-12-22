---
title: 二阶级联滤波器(SOS)
author: lixinghui
date: 2023-11-29 14:10:00 +0800
categories: [DSP, filter]
tags: [数字信号处理]
math: true
---

二阶滤波器是二阶（两个极点和两个零点）的IIR滤波器。考虑到更高阶数的滤波器对系数敏感，二阶可以单独使用，或者在更复杂的滤波器中作为基本的构建单元。

二阶滤波器的传递函数为：

$$
H(z)=\frac{b_0+b_1z^{-1}+b_2z^{-2}}{a_0+a_1z^{-1}+a_2z^{-2}}
$$

其中参数$$a_0=1$$;

## 直接I型

二阶滤波器有几种不同的结构，最直观也最直接的实现是用一个二阶差分方程：

$$
y[n]=b_0x[n]+b_1x[n-1]+b_2x[n-2]-a_1y[n-1]-a_2y[n-2]
$$

针对b参数加入G参数，设置$$b=G*b^{'}$$:

$$
y[n]=G*(b_0^{'}x[n]+b_1^{'}x[n-1]+b_2^{'}x[n-2])-a_1y[n-1]-a_2y[n-2]
$$

在这里$$b_0,b_1,b_2$$为零点，$$a_1,a_2$$为极点。它的结构为：

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c3/Biquad_filter_DF-I.svg/400px-Biquad_filter_DF-I.svg.png)

在定点（没有浮点数）处理器中，Direct Form I是最好实现的，因为只有一个加法点（定点DSP一般都有扩展的累加器，可以在运算中有一定的溢出）。

## 直接II型

直接II型实现了直接I型相同的归一化函数，但是结构不同，分为了两个部分，计算方式为：

$$
y[n]=b_0 \omega[n]+b_1 \omega[n-1]+b_2 \omega[n-2]
$$

其中：

$$
\omega[n]=x[n]-a_1 \omega[n-1]-a_2 \omega[n-2]
$$

它的结构为：

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/Biquad_filter_DF-IIx.svg/400px-Biquad_filter_DF-IIx.svg.png)

直接形式2的实现只需要N个延迟单元，其中N是滤波器的阶数——可能是直接形式1的一半。缺点是直接形式2增加了高Q滤波器或共振滤波器算术溢出的可能性。[1]已经证明，随着Q的增加，两种直接形式拓扑的舍入噪声都无界地增加。

直接形式2的实现被称为规范形式，因为它使用了最少的延迟、加法器和乘法器，产生的传递函数与直接形式1的实现相同。



还有转置的I型和II型，这里就不说了，具体可以参考[Digital_biquad_filter](https://en.wikipedia.org/wiki/Digital_biquad_filter)



## 总结

总结，direct form I更适合定点计算，转置(transposed) 或Direct form II更适合浮点计算。

量化误差，在低频情况下，容易导致双二阶滤波器更脆弱（不稳定），主要是因为反馈系数(b1和b2)以及中间值(the delay memory)。精度不够，会导致不容易精确控制极点的位置，尤其在极点靠近单位圆的情况下，因此，量化误差会变成一个重要问题。第二个问题，是关于中间值的，相乘之后会产生更多的位，当保存下来时，需要做截断处理。这个误差反馈回滤波器系统，可能导致系统不稳定。32位浮点数对于音频滤波器来讲一般是足够好了。但是，对于很低频率（如控制类信号的滤波）和高采样率情况下，可能需要双精度才行。

对于定点实现的滤波器，24位的系数和中间存储，基本上可以很好的满足需要了，但是48k采样率下从300Hz往下（或者96k采样率下从600Hz往下），就开始变得不稳定了。定点处理器上实现双精度开销很大，幸好，有一个简单的方法可以提高稳定度。看看Direct Form I的结构图，把高精度的累加结果保存在低精度的中间存储器中，这个过程会造成精度损失。把精度损失（量化损失：高精度计算结果与低精度存储之间的差值）加回到下一次输出结果的计算中，近似达到双精度，当然，会增加一些运算量。这个技巧被称为First Order Noise Shaping. 有更高阶的Noise Shapers，但是First Order已经足够好了，可以应付所有音频需要，即使在高采样率的情况下。

**定点计算建议使用I型，浮点计算建议使用II型。**

## C语言参考代码

```c
void DirectI_Filter(int *x, int *y, int len, double *b, double *a)
{
    if ((x == NULL) || (y == NULL) || (b == NULL) || (a == NULL))
    {
        return;
    }
    static double hisx[3] = {0};
    static double hisy[3] = {0};
    double sum;
    int i;
    for (i = 0; i < len; i++)
    {
        hisx[0] = x[i];
        sum = b[0] * hisx[0] + b[1] * hisx[1] + b[2] * hisx[2];
        sum = sum - a[1] * hisy[1] - a[2] * hisy[2];
        hisy[0] = sum;

        hisx[2] = hisx[1];
        hisx[1] = hisx[0];
        hisy[2] = hisy[1];
        hisy[1] = hisy[0];

        y[i] = hisy[0];
    }
}
```

```c
void DirectII_Filter(int *x, int *y, int len, double *b, double *a)
{
    if ((x == NULL) || (y == NULL) || (b == NULL) || (a == NULL))
    {
        return;
    }
    static double hisw[3] = {0};
    int i;
    for (i = 0; i < len; i++)
    {
        hisw[0] = x[i] - a[1] * hisw[1] - a[2] * hisw[2];
        y[i] = b[0] * hisw[0] + b[1] * hisw[1] + b[2] * hisw[2];

        hisw[2] = hisw[1];
        hisw[1] = hisw[0];
    }
}
```