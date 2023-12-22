---
title: 一种二阶滤波器的设计方式
author: lixinghui
date: 2023-10-13 14:10:00 +0800
categories: [DSP, filter]
tags: [数字信号处理]
math: true
render_with_liquid: false
---

参考DAFX_Digtial_Audio_Effects这本书，一个二阶滤波器的设计为：

## 公式

|              | b0                          | b1                           | b2                          | a1                           | a2                          |
| ------------ | --------------------------- | ---------------------------- | --------------------------- | ---------------------------- | --------------------------- |
| `Lowpass`    | $\frac{K^2Q}{K^2Q+K+Q}$     | $\frac{2K^2Q}{K^2Q+K+Q}$     | $\frac{K^2Q}{K^2Q+K+Q}$     | $\frac{2Q(K^2-1)}{K^2Q+K+Q}$ | $\frac{K^2Q-K+Q}{K^2Q+K+Q}$ |
| `Highpass`   | $\frac{Q}{K^2Q+K+Q}$        | $-\frac{2Q}{K^2Q+K+Q}$       | $\frac{Q}{K^2Q+K+Q}$        | $\frac{2Q(K^2-1)}{K^2Q+K+Q}$ | $\frac{K^2Q-K+Q}{K^2Q+K+Q}$ |
| `Bandpass`   | $\frac{K}{K^2Q+K+Q}$        | $0$                          | $-\frac{K}{K^2Q+K+Q}$       | $\frac{2Q(K^2-1)}{K^2Q+K+Q}$ | $\frac{K^2Q-K+Q}{K^2Q+K+Q}$ |
| `Bandreject` | $\frac{Q(1+K^2)}{K^2Q+K+Q}$ | $\frac{2Q(K^2-1)}{K^2Q+K+Q}$ | $\frac{Q(1+K^2)}{K^2Q+K+Q}$ | $\frac{2Q(K^2-1)}{K^2Q+K+Q}$ | $\frac{K^2Q-K+Q}{K^2Q+K+Q}$ |
| `Allpass`    | $\frac{K^2Q-K+Q}{K^2Q+K+Q}$ | $\frac{2Q(K^2-1)}{K^2Q+K+Q}$ | $1$                         | $\frac{2Q(K^2-1)}{K^2Q+K+Q}$ | $\frac{K^2Q-K+Q}{K^2Q+K+Q}$ |

>   $K=tan(\pi fc/fs) \qquad fc为中心频率，fs为采样率$
>
>   对于低通滤波器和高通滤波器，当$Q=\frac{1}{\sqrt{2}}$，滤波器最大平坦到截至频率，对于较低的Q，有着较大的通带衰减，对于较高的Q，在Fc的地方幅度会上升。
>
>   对于带通和带阻滤波器，Q与带宽有关，$Q=\frac{f_c}{f_b}$,与带宽是反相关的。
>
>   这里会在设定的中心频率和带宽之间-3dB的增益。

## matlab代码

```matlab
function [b,a] = Lowpass(K, Q)

b0=K^2*Q/(K^2*Q+K+Q);
b1=2*K^2*Q/(K^2*Q+K+Q);
b2=K^2*Q/(K^2*Q+K+Q);

a0=1;
a1=2*Q*(K^2-1)/(K^2*Q+K+Q);
a2=(K^2*Q-K+Q)/(K^2*Q+K+Q);

b=[b0,b1,b2];
a=[a0,a1,a2];
end


function [b,a] = Highpass(K, Q)

b0=Q/(K^2*Q+K+Q);
b1=-2*Q/(K^2*Q+K+Q);
b2=Q/(K^2*Q+K+Q);

a0=1;
a1=2*Q*(K^2-1)/(K^2*Q+K+Q);
a2=(K^2*Q-K+Q)/(K^2*Q+K+Q);

b=[b0,b1,b2];
a=[a0,a1,a2];

end


function [b,a] = Bandpass(K, Q)

b0=K/(K^2*Q+K+Q);
b1=0;
b2=-K/(K^2*Q+K+Q);

a0=1;
a1=2*Q*(K^2-1)/(K^2*Q+K+Q);
a2=(K^2*Q-K+Q)/(K^2*Q+K+Q);

b=[b0,b1,b2];
a=[a0,a1,a2];

end


function [b,a] = Bandreject(K, Q)

b0=(Q*(1+K^2))/(K^2*Q+K+Q);
b1=(2*Q*(K^2-1))/(K^2*Q+K+Q);
b2=(Q*(1+K^2))/(K^2*Q+K+Q);

a0=1;
a1=2*Q*(K^2-1)/(K^2*Q+K+Q);
a2=(K^2*Q-K+Q)/(K^2*Q+K+Q);

b=[b0,b1,b2];
a=[a0,a1,a2];

end
```

## C语言代码

```c
void Lowpass(double K, double Q, double *b, double *a)
{
    if ((b == NULL) || (a == NULL))
    {
        return;
    }
    double T = K * K * Q + K + Q;

    b[0] = K * K * Q / T;
    b[1] = 2 * K * K * Q / T;
    b[2] = K * K * Q / T;

    a[0] = 1;
    a[1] = 2 * Q * (K * K - 1) / T;
    a[2] = (K * K * Q - K + Q) / T;
}

void Highpass(double K, double Q, double *b, double *a)
{
    if ((b == NULL) || (a == NULL))
    {
        return;
    }
    double T = K * K * Q + K + Q;

    b[0] = Q / T;
    b[1] = -2 * Q / T;
    b[2] = Q / T;

    a[0] = 1;
    a[1] = 2 * Q * (K * K - 1) / T;
    a[2] = (K * K * Q - K + Q) / T;
}

void Bandpass(double K, double Q, double *b, double *a)
{
    if ((b == NULL) || (a == NULL))
    {
        return;
    }
    double T = K * K * Q + K + Q;

    b[0] = K / T;
    b[1] = 0;
    b[2] = -K / T;

    a[0] = 1;
    a[1] = 2 * Q * (K * K - 1) / T;
    a[2] = (K * K * Q - K + Q) / T;
}

void Bandreject(double K, double Q, double *b, double *a)
{
    if ((b == NULL) || (a == NULL))
    {
        return;
    }
    double T = K * K * Q + K + Q;

    b[0] = Q * (1 + K * K) / T;
    b[1] = 2 * Q * (K * 2 - 1) / T;
    b[2] = Q * (1 + K * K) / T;

    a[0] = 1;
    a[1] = 2 * Q * (K * K - 1) / T;
    a[2] = (K * K * Q - K + Q) / T;
}
```





---



## 二阶滤波器的使用

二阶滤波器可以分为直接I型和直接II型

直接I型为：

$$
y[n]=b_0x[n]+b_1x[n-1]+b_2x[n-2]-a_1y[n-1]-a_2y[n-2]
$$

针对b参数加入G参数，设置$$b=G*b^{'}$$:

$$
y[n]=G*(b_0^{'}x[n]+b_1^{'}x[n-1]+b_2^{'}x[n-2])-a_1y[n-1]-a_2y[n-2]
$$

直接II型为：

$$
y[n]=b_0w[n]+b_1w[n-1]+b_2w[n-2] 
$$

$$
w[n]=x[n]-a_1w[n-1]-a_2w[n-2]
$$

**定点计算建议使用I型，浮点计算建议使用II型。**

C语言参考代码：

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

## 参考链接：

[Digital biquad filter](https://en.wikipedia.org/wiki/Digital_biquad_filter#Direct_form_1)