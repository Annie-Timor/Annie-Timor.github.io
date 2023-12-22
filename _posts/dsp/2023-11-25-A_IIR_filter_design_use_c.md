---
title: 一种IIR滤波器生成方式-C语言
author: lixinghui
date: 2023-11-25 11:10:00 +0800
categories: [DSP, filter]
tags: [数字信号处理]
---



通过C语言直接生成巴特沃斯滤波器和切比雪夫II型滤波器

>   这里生成的低通和高通滤波器是二阶级联的也就是SOS(second-order sections)格式的，带通和带阻滤波器是四阶级联的。
{: .prompt-tip }

如果需要将二阶或者四阶的滤波器串起来可以使用卷积来完成，如下：

```c
void convolution(const double *signal, int signalLength, const double *kernel, int kernelLength, double *output)
{
    int outputLength = signalLength + kernelLength - 1; // 计算输出数组长度

    // 初始化输出数组为0
    for (int i = 0; i < outputLength; i++)
    {
        output[i] = 0.0;
    }

    // 执行卷积计算
    for (int i = 0; i < signalLength; i++)
    {
        for (int j = 0; j < kernelLength; j++)
        {
            output[i + j] += signal[i] * kernel[j];
        }
    }
}
```



## filter_design.h

```c
#ifndef FILTER_DESIGN_H
#define FILTER_DESIGN_H

typedef enum {
    LowpassFilter=1,
    HighpassFilter,
    BandpassFilter,
    BandstopFilter,
}FilterType;

void iirbcf(int ifilt, int band, int ns, int n, double f1, double f2, double f3, double f4, double db, double *b, double *a);

void create_bw_lpf(double fc, double fs, int ns, double *b, double *a);
void create_bw_hpf(double fc, double fs, int ns, double *b, double *a);
void create_bw_bpf(double flc, double fhc, double fs, int ns, double *b, double *a);
void create_bw_bsf(double flc, double fhc, double fs, int ns, double *b, double *a);
void create_che_lpf(double fr, double fs, int ns, double db, double *b, double *a);
void create_che_hpf(double fr, double fs, int ns, double db, double *b, double *a);
void create_che_bpf(double flr, double fhr, double fs, int ns, double db, double *b, double *a);
void create_che_bsf(double flr,  double fhr, double fs, int ns, double db, double *b, double *a);
#endif // !FILTER_DESIGN_H

```



## filter_design.c

```c
#include "filter_design.h"
#include <math.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static double cosh1(double x);
static double warp(double f);
static double bpsub(double om, double fh, double fl);
static double omin(double om1, double om2);
static void bwtf(int ln, int k, int n, double *d, double *c);
static void chebyi(int ln, int k, int n, double ep, double *d, double *c);
static void chebyii(int ln, int k, int n, double ws, double att, double *d, double *c);
static void fblt(double *d, double *c, int n, int band, double fln, double fhn, double *b, double *a);
static double combin(int i1, int i2);
static void bilinear(double *d, double *c, double *b, double *a, int n);

/**
 * @brief
 * @param 巴特沃兹滤波器：
 * @param 低通时，f1是通带边界频率，f2=f3=f4=0;
 * @param 高通时，f2是通带边界频率，f1=f3=f4=0
 * @param 带通时，f2是通带下边界频率，f3是通带上边界频率，f1=f4=0;
 * @param 带阻时，f1是通带下边界频率，f4是通带上边界频率，f2=f3=0
 * @param 切比雪夫滤波器：
 * @param 低通时，f1是通带边界频率，f2是阻带边界频率，f3=f4=0;
 * @param 高通时，f2是通带边界频率，f1是阻带边界频率，f3=f4=0;
 * @param 带通时，f2是通带下边界频率，f3是通带上边界频率，f1是阻带下边界频率，f4是阻带上边界频率；
 * @param 带阻时，f1是通带下边界频率，f4是通带上边界频率，f2是阻带下边界频率，f3是阻带上边界频率。
 *
 * @param ifilt 整型变量。滤波器的类型。取值为1、2和3，分别对应切比雪夫、逆切比雪夫和巴特沃兹滤波器。
 * @param band 整型变量。滤波器的通带形式。取值为1、2、3和4，分别对应低通、高通、带通和带阻滤波器。
 * @param ns 整型变量。滤波器的n阶节数。
 * @param n 整型变量。滤波器每节的阶数。对于低通和高通滤波器，n=2;对于带通和带阻滤波器，n=4。
 * @param f1 双精度实型变量。
 * @param f2 双精度实型变量。
 * @param f3 双精度实型变量。
 * @param f4 双精度实型变量。
 * @param db 双精度实型变量。滤波器的阻带衰减（用dB表示）。
 * @param b 双精度实型二维数组，体积为ns*(n+1) 存放滤波器分子多项式的系数。b[j][i]表示第j个n阶节的分子多项式的第i个系数。
 * @param a 双精度实型二维数组，体积为ns*(n+1) 存放滤波器分母多项式的系数。a[j][i]表示第j个n阶节的分母多项式的第i个系数。
 */
void iirbcf(int ifilt, int band, int ns, int n, double f1, double f2, double f3, double f4, double db, double *b, double *a)
{
    int k;
    double omega, lambda, epslon, fl, fh;
    double d[5], c[5];

    if ((band == 1) || (band == 4))
    {
        fl = f1;
    }
    else if ((band == 2) || (band == 3))
    {
        fl = f2;
    }

    if (band <= 3)
    {
        fh = f3;
    }
    else if (band == 4)
    {
        fh = f4;
    }

    if (ifilt < 3)
    {
        switch (band)
        {
        case 1:
        case 2:
        {
            omega = warp(f2) / warp(f1);
            break;
        }
        case 3:
        {
            omega = omin(bpsub(warp(f1), fh, fl), bpsub(warp(f4), fh, fl));
            break;
        }
        case 4:
        {
            omega = omin(1.0 / bpsub(warp(f2), fh, fl), 1.0 / bpsub(warp(f3), fh, fl));
        }
        }
        lambda = pow(10.0, (db / 20.0));
        epslon = lambda / cosh(2 * ns * cosh1(omega));
    }

    for (k = 0; k < ns; k++)
    {
        switch (ifilt)
        {
        case 1:
        {
            chebyi(2 * ns, k, 4, epslon, d, c);
            break;
        }
        case 2:
        {
            chebyii(2 * ns, k, 4, omega, lambda, d, c);
            break;
        }
        case 3:
        {
            bwtf(2 * ns, k, 4, d, c);
            break;
        }

        default:
            break;
        }
        fblt(d, c, n, band, fl, fh, &b[k * (n + 1) + 0], &a[k * (n + 1) + 0]);
    }
}

static double cosh1(double x)
{
    double z;
    z = log(x + sqrt(x * x - 1.0));
    return (z);
}

static double warp(double f)
{
    double z;
    z = tan(M_PI * f);
    return (z);
}

static double bpsub(double om, double fh, double fl)
{
    double z;
    z = (om * om - warp(fh) * warp(fl)) / ((warp(fh) - warp(fl)) * om);
    return (z);
}

static double omin(double om1, double om2)
{
    double z, z1, z2;
    z1 = fabs(om1);
    z2 = fabs(om2);
    z = (z1 < z2) ? z1 : z2;
    return (z);
}

static void bwtf(int ln, int k, int n, double *d, double *c)
{
    int i;
    double tmp;
    d[0] = 1.0;
    c[0] = 1.0;
    for (i = 1; i <= n; i++)
    {
        d[i] = 0.0;
        c[i] = 0.0;
    }
    tmp = (k + 1) - (ln + 1.0) / 2.0;
    if (tmp == 0.0)
    {
        c[1] = 1.0;
    }
    else
    {
        c[1] = -2.0 * cos((2 * (k + 1) + ln - 1) * M_PI / (2 * ln));
        c[2] = 1.0;
    }
}

static void chebyi(int ln, int k, int n, double ep, double *d, double *c)
{
    int i;
    double gamma, omega, sigma;

    gamma = pow(((1.0 + sqrt(1.0 + ep * ep)) / ep), 1.0 / ln);
    sigma = 0.5 * (1.0 / gamma - gamma) * sin((2 * (k + 1) - 1) * M_PI / (2 * ln));
    omega = 0.5 * (1.0 / gamma + gamma) * cos((2 * (k + 1) - 1) * M_PI / (2 * ln));

    for (i = 0; i <= n; i++)
    {
        d[i] = 0.0;
        c[i] = 0.0;
    }
    if (((ln % 2) == 1) && ((k + 1) == (ln + 1) / 2))
    {
        d[0] = -sigma;
        c[0] = d[0];
        c[1] = 1.0;
    }
    else
    {
        c[0] = sigma * sigma + omega * omega;
        c[1] = -2.0 * sigma;
        c[2] = 1.0;
        d[0] = c[0];
        if (((ln % 2) == 0) && (k == 0))
        {
            d[0] = d[0] / sqrt(1.0 + ep * ep);
        }
    }
}

static void chebyii(int ln, int k, int n, double ws, double att, double *d, double *c)
{
    int i;
    double gamma, alpha, beta, sigma, omega, scln, scld;

    gamma = pow((att + sqrt(att * att - 1.0)), 1.0 / ln);
    alpha = 0.5 * (1.0 / gamma - gamma) * sin((2 * (k + 1) - 1) * M_PI / (2 * ln));
    beta = 0.5 * (1.0 / gamma + gamma) * cos((2 * (k + 1) - 1) * M_PI / (2 * ln));
    sigma = ws * alpha / (alpha * alpha + beta * beta);
    omega = -1.0 * ws * beta / (alpha * alpha + beta * beta);
    for (i = 0; i <= n; i++)
    {
        d[i] = 0.0;
        c[i] = 0.0;
    }
    if (((ln % 2) == 1) && ((k + 1) == (ln + 1) / 2))
    {
        d[0] = -1.0 * sigma;
        c[0] = d[0];
        c[1] = 1.0;
    }
    else
    {
        scln = sigma * sigma + omega * omega;
        scld = pow((ws / cos((2 * (k + 1) - 1) * M_PI / (2 * ln))), 2);
        d[0] = scln * scld;
        d[2] = scln;
        c[0] = d[0];
        c[1] = -2.0 * sigma * scld;
        c[2] = scld;
    }
}

static void fblt(double *d, double *c, int n, int band, double fln, double fhn, double *b, double *a)
{
    int i, k, m, n1, n2, ls;
    double w, w0, w1, w2, tmp, tmpd, tmpc, *work;

    w1 = tan(M_PI * fln);
    for (i = n; i >= 0; i--)
    {
        if ((c[i] != 0.0) || (d[i] != 0.0))
        {
            break;
        }
    }
    m = i;
    switch (band)
    {
    case 1:
    case 2:
    {
        n2 = m;
        n1 = n2 + 1;
        if (band == 2)
        {
            for (i = 0; i <= m / 2; i++)
            {
                tmp = d[i];
                d[i] = d[m - i];
                d[m - i] = tmp;
                tmp = c[i];
                c[i] = c[m - i];
                c[m - i] = tmp;
            }
        }
        for (i = 0; i <= m; i++)
        {
            d[i] = d[i] / pow(w1, i);
            c[i] = c[i] / pow(w1, i);
        }
        break;
    }

    case 3:
    case 4:
    {
        n2 = 2 * m;
        n1 = n2 + 1;
        work = malloc(n1 * n1 * sizeof(double));
        w2 = tan(M_PI * fhn);
        w = w2 - w1;
        w0 = w1 * w2;
        if (band == 4)
        {
            for (i = 0; i < m / 2; i++)
            {
                tmp = d[i];
                d[i] = d[m - i];
                d[m - i] = tmp;
                tmp = c[i];
                c[i] = c[m - i];
                c[m - i] = tmp;
            }
        }
        for (i = 0; i <= n2; i++)
        {
            work[0 * n1 + i] = 0.0;
            work[1 * n1 + i] = 0.0;
        }
        for (i = 0; i <= m; i++)
        {
            tmpd = d[i] * pow(w, (m - i));
            tmpc = c[i] * pow(w, (m - i));
            for (k = 0; k <= i; k++)
            {
                ls = m + i - 2 * k;
                tmp = combin(i, i) / (combin(k, k) * combin(i - k, i - k));
                work[0 * n1 + ls] += tmpd * pow(w0, k) * tmp;
                work[1 * n1 + ls] += tmpc * pow(w0, k) * tmp;
            }
        }
        for (i = 0; i <= n2; i++)
        {
            d[i] = work[0 * n1 + i];
            c[i] = work[1 * n1 + i];
        }
        free(work);
        break;
    }

    default:
        break;
    }
    bilinear(d, c, b, a, n);
}

static double combin(int i1, int i2)
{
    int i;
    double s;
    s = 1.0;
    if (i2 == 0)
    {
        return (s);
    }
    for (i = i1; i > (i1 - i2); i--)
    {
        s *= i;
    }
    return (s);
}

static void bilinear(double *d, double *c, double *b, double *a, int n)
{
    int i, j, n1;
    double sum, atmp, scale, *temp;
    n1 = n + 1;
    temp = malloc(n1 * n1 * sizeof(double));
    for (j = 0; j <= n; j++)
    {
        temp[j * n1 + 0] = 1.0;
    }
    sum = 1.0;
    for (i = 1; i <= n; i++)
    {
        sum = sum * (double)(n - i + 1) / (double)i;
        temp[0 * n1 + i] = sum;
    }
    for (i = 1; i <= n; i++)
    {
        for (j = 1; j <= n; j++)
        {
            temp[j * n1 + i] = temp[(j - 1) * n1 + i] - temp[j * n1 + i - 1] - temp[(j - 1) * n1 + i - 1];
        }
    }
    for (i = n; i >= 0; i--)
    {
        b[i] = 0.0;
        atmp = 0.0;
        for (j = 0; j <= n; j++)
        {
            b[i] = b[i] + temp[j * n1 + i] * d[j];
            atmp = atmp + temp[j * n1 + i] * c[j];
        }
        scale = atmp;
        if (i != 0)
        {
            a[i] = atmp;
        }
    }
    for (i = 0; i <= n; i++)
    {
        b[i] = b[i] / scale;
        a[i] = a[i] / scale;
    }
    a[0] = 1.0;
    free(temp);
}

/**
 * @brief Create a butterworth lowpass filter
 *
 * @param fc passband frequency
 * @param fs sample frequency
 * @param ns number of sections
 * @param b coefficients b
 * @param a coefficients a
 */
void create_bw_lpf(double fc, double fs, int ns, double *b, double *a)
{
    int ifilt = 3;
    FilterType type = LowpassFilter;
    int n = (type <= HighpassFilter) ? 2 : 4;
    double f1, f2, f3, f4;
    double db = 40;

    fc = fc / fs; /*归一化处理*/
    f1 = f2 = f3 = f4 = 0;
    f1 = fc;
    iirbcf(ifilt, type, ns, n, f1, f2, f3, f4, db, b, a);
}

/**
 * @brief Create a butterworth highpass filter
 *
 * @param fc passband frequency
 * @param fs sample frequency
 * @param ns number of sections
 * @param b coefficients b
 * @param a coefficients a
 */
void create_bw_hpf(double fc, double fs, int ns, double *b, double *a)
{
    int ifilt = 3;
    FilterType type = HighpassFilter;
    int n = (type <= HighpassFilter) ? 2 : 4;
    double f1, f2, f3, f4;
    double db = 40;

    fc = fc / fs; /*归一化处理*/
    f1 = f2 = f3 = f4 = 0;
    f2 = fc;
    iirbcf(ifilt, type, ns, n, f1, f2, f3, f4, db, b, a);
}

/**
 * @brief Create a butterworth bandpass filter
 *
 * @param flc low passband frequency
 * @param fhc high passband frequency
 * @param fs sample frequency
 * @param ns number of sections
 * @param b coefficients b
 * @param a coefficients a
 */
void create_bw_bpf(double flc, double fhc, double fs, int ns, double *b, double *a)
{
    int ifilt = 3;
    FilterType type = BandpassFilter;
    int n = (type <= HighpassFilter) ? 2 : 4;
    double f1, f2, f3, f4;
    double db = 40;

    flc = flc / fs; /*归一化处理*/
    fhc = fhc / fs;
    f1 = f2 = f3 = f4 = 0;
    f2 = flc;
    f3 = fhc;
    iirbcf(ifilt, type, ns, n, f1, f2, f3, f4, db, b, a);
}

/**
 * @brief Create a butterworth bandstop filter
 *
 * @param flc low passband frequency
 * @param fhc high passband frequency
 * @param fs sample frequency
 * @param ns number of sections
 * @param b coefficients b
 * @param a coefficients a
 */
void create_bw_bsf(double flc, double fhc, double fs, int ns, double *b, double *a)
{
    int ifilt = 3;
    FilterType type = BandstopFilter;
    int n = (type <= HighpassFilter) ? 2 : 4;
    double f1, f2, f3, f4;
    double db = 40;

    flc = flc / fs; /*归一化处理*/
    fhc = fhc / fs;
    f1 = f2 = f3 = f4 = 0;
    f1 = flc;
    f4 = fhc;
    iirbcf(ifilt, type, ns, n, f1, f2, f3, f4, db, b, a);
}

/**
 * @brief Create a Chebyshev lowpass filter
 *
 * @param fr stopband frequency
 * @param fs sample frequency
 * @param ns number of sections
 * @param db stopband gain
 * @param b coefficients b
 * @param a coefficients a
 */
void create_che_lpf(double fr, double fs, int ns, double db, double *b, double *a)
{
    int ifilt = 2;
    FilterType type = LowpassFilter;
    int n = (type <= HighpassFilter) ? 2 : 4;
    double f1, f2, f3, f4;

    fr = fr / fs; /*归一化处理*/
    f1 = f2 = f3 = f4 = 0;
    f1 = fr - 0.01;
    f2 = fr;
    iirbcf(ifilt, type, ns, n, f1, f2, f3, f4, db, b, a);
}

/**
 * @brief Create a Chebyshev highpass filter
 *
 * @param fr stopband frequency
 * @param fs sample frequency
 * @param ns number of sections
 * @param db stopband gain
 * @param b coefficients b
 * @param a coefficients a
 */
void create_che_hpf(double fr, double fs, int ns, double db, double *b, double *a)
{
    int ifilt = 2;
    FilterType type = HighpassFilter;
    int n = (type <= HighpassFilter) ? 2 : 4;
    double f1, f2, f3, f4;

    fr = fr / fs; /*归一化处理*/
    f1 = f2 = f3 = f4 = 0;
    f1 = fr;
    f2 = fr + 0.01;
    iirbcf(ifilt, type, ns, n, f1, f2, f3, f4, db, b, a);
}

/**
 * @brief Create a Chebyshev bandpass filter
 *
 * @param flr low stopband frequency
 * @param fhr high stopband frequency
 * @param fs sample frequency
 * @param ns number of sections
 * @param db stopband gain
 * @param b coefficients b
 * @param a coefficients a
 */
void create_che_bpf(double flr, double fhr, double fs, int ns, double db, double *b, double *a)
{
    int ifilt = 2;
    FilterType type = BandpassFilter;
    int n = (type <= HighpassFilter) ? 2 : 4;
    double f1, f2, f3, f4;

    flr = flr / fs; /*归一化处理*/
    fhr = fhr / fs;
    f1 = f2 = f3 = f4 = 0;
    f1 = flr;
    f2 = flr + 0.01;
    f3 = fhr - 0.01;
    f4 = fhr;
    iirbcf(ifilt, type, ns, n, f1, f2, f3, f4, db, b, a);
}

/**
 * @brief Create a Chebyshev bandstop filter
 *
 * @param flr low stopband frequency
 * @param fhr high stopband frequency
 * @param fs sample frequency
 * @param ns number of sections
 * @param db stopband gain
 * @param b coefficients b
 * @param a coefficients a
 */
void create_che_bsf(double flr, double fhr, double fs, int ns, double db, double *b, double *a)
{
    int ifilt = 2;
    FilterType type = BandstopFilter;
    int n = (type <= HighpassFilter) ? 2 : 4;
    double f1, f2, f3, f4;

    flr = flr / fs; /*归一化处理*/
    fhr = fhr / fs;
    f1 = f2 = f3 = f4 = 0;
    f1 = flr - 0.01;
    f2 = flr;
    f3 = fhr;
    f4 = fhr + 0.01;
    iirbcf(ifilt, type, ns, n, f1, f2, f3, f4, db, b, a);
}
```



## main.c

```c
#include <math.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "filter_design.h"

int main(int argc, char **argv)
{
    int i, k, ns;
    double a[50], b[50];

    ns = 2;
    create_che_bsf(0.2,0.3,1,ns,40,b,a);
    printf("<---------------------------------------->\n");
    // for (i = 0; i < ns; i++)
    // {
    //     printf("b%d=[%f,%f,%f];\n", i, b[3 * i + 0], b[3 * i + 1], b[3 * i + 2]);
    //     printf("a%d=[%f,%f,%f];\n", i, a[3 * i + 0], a[3 * i + 1], a[3 * i + 2]);
    // }
    for (i = 0; i < ns; i++)
    {
        printf("b%d=[%f,%f,%f,%f,%f];\n", i, b[5 * i + 0], b[5 * i + 1], b[5 * i + 2], b[5 * i + 3], b[5 * i + 4]);
        printf("a%d=[%f,%f,%f,%f,%f];\n", i, a[5 * i + 0], a[5 * i + 1], a[5 * i + 2], a[5 * i + 3], a[5 * i + 4]);
    }
    printf("<---------------------------------------->\n\n");
    return 0;
}
```

