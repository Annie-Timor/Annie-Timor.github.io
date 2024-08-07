---
title: 基于滤波器的EQ均衡器设计
author: lixinghui
date: 2023-8-2 14:10:00 +0800
categories: [Speech, EQ]
tags: [语音杂谈]
---



# 探索音频中的EQ(均衡器)

均衡器(EQ)是音频处理中至关重要的工具，用于调节不同频率的音量，从而优化声音的整体效果。本文将详细介绍EQ的工作原理、类型、应用及常见设置方法。

## 什么是均衡器？

均衡器(EQ)是一种音频处理器，它可以提高或减少特定频率范围内的音频信号强度。均衡器通常用于：

- 调整音频的音色
- 补偿音频系统的缺陷
- 创造特殊音效

## 均衡器的工作原理

EQ通过调整音频信号中的频率成分来改变声音的特性。频率范围通常分为低频、中频和高频，均衡器可以分别调节这些频段的增益（音量）。

### 频率范围

1. **低频（20Hz-250Hz）**：影响低音和厚重感。
2. **中频（250Hz-4kHz）**：影响人声和大多数乐器的主要频率范围。
3. **高频（4kHz-20kHz）**：影响清晰度和亮度。

## 均衡器的类型

### 1. 图示均衡器（Graphic EQ）

图示均衡器通过固定频段的滑块来调整每个频段的增益。这种均衡器的优点是直观、易用，常见于现场音响系统和一些简单的音频处理软件中。

### 2. 参数均衡器（Parametric EQ）

参数均衡器提供更灵活的控制，允许用户调节频率、增益和Q值（频段宽度）。这种均衡器广泛应用于录音和混音过程中，适用于精细调整音频。

### 3. 半参数均衡器（Semi-Parametric EQ）

半参数均衡器介于图示均衡器和参数均衡器之间，通常允许调节频率和增益，但Q值是固定的。

## 均衡器的应用

### 1. 调整音色

均衡器可以用来增强或削弱特定频段，从而调整整体音色。例如，增加低频可以使声音更丰满，增加高频可以提高清晰度。

### 2. 消除不必要的频率

录音过程中常会捕捉到一些不需要的频率，通过EQ可以削减这些频率，使音频更干净。

### 3. 补偿音响设备的不足

不同的音响设备有不同的频率响应，通过EQ可以补偿这些设备的不足，达到理想的音频效果。

### 4. 创造特殊音效

均衡器还可以用于创造特效，比如电话效果、老式唱片效果等。

## 常见的EQ设置

### 人声处理

- 削减低频（低于80Hz），以减少低频噪音。
- 增强中频（1kHz-3kHz），使人声更清晰。
- 增加高频（8kHz-12kHz），提高亮度。

### 吉他处理

- 削减低频（低于100Hz），减少不需要的低音。
- 增强中频（2kHz-5kHz），使吉他更突出。
- 适当增加高频（6kHz-8kHz），增加清晰度。

### 鼓处理

- 增强低频（60Hz-100Hz），增加低音鼓的冲击力。
- 增强中高频（2kHz-5kHz），使军鼓更清脆。

## 结论

均衡器是音频处理中不可或缺的工具，通过合理的EQ设置，可以显著提升音频质量。在使用均衡器时，需根据具体需求进行调整，避免过度调整导致音频失真。

---


## CODE:

### eq_filterdesigner.h

```c
#ifndef EQ_FILTERDESIGNER_H
#define EQ_FILTERDESIGNER_H

void PeakBoost(double fc, double fb, double fs, double G, double *b, double *a);
void PeakCut(double fc, double fb, double fs, double G, double *b, double *a);
void ShelvingLF_Boost(double fc, double fb, double fs, double G, double *b, double *a);
void ShelvingLF_Cut(double fc, double fb, double fs, double G, double *b, double *a);
void ShelvingHF_Boost(double fc, double fb, double fs, double G, double *b, double *a);
void ShelvingHF_Cut(double fc, double fb, double fs, double G, double *b, double *a);

void Gainc(double *b, double *a, int n, int ns, double *x, double *y, int len, int sign);

#endif // EQ_FILTERDESIGNER_H
```

### eq_filterdesigner.c

```c
#include "eq_filterdesigner.h"
#include <stdio.h>
#include <string.h>
#include <math.h>


/**
 * @brief This function calculates the coefficients for a  peak Boost filter.
 * @param fc The center frequency
 * @param fb The bandwidth
 * @param fs The sampling frequency.
 * @param G  The gain in dB.
 * @param b  A pointer to an array of size 3, where the calculated coefficients for the numerator of the filter will be stored.
 * @param a  A pointer to an array of size 3, where the calculated coefficients for the denominator of the filter will be stored.
 * @return void
 */
void PeakBoost(double fc, double fb, double fs, double G, double *b, double *a)
{
    double V0 = pow(10, G / 20);
    double K = tan(M_PI * fc / fs);
    double Q = fc / fb;

    double den = 1 + K / Q + K * K;

    b[0] = (1 + V0 / Q * K + K * K) / den;
    b[1] = (2 * K * K - 2) / den;
    b[2] = (1 - V0 / Q * K + K * K) / den;

    a[0] = 1;
    a[1] = (2 * K * K - 2) / den;
    a[2] = (1 - K / Q + K * K) / den;
}

/**
 * @brief This function calculates the coefficients for a  peak cut filter.
 * @param fc The center frequency
 * @param fb The bandwidth
 * @param fs The sampling frequency.
 * @param G  The gain in dB.
 * @param b  A pointer to an array of size 3, where the calculated coefficients for the numerator of the filter will be stored.
 * @param a  A pointer to an array of size 3, where the calculated coefficients for the denominator of the filter will be stored.
 * @return void
 */
void PeakCut(double fc, double fb, double fs, double G, double *b, double *a)
{
    double V0 = pow(10, G / 20);
    double K = tan(M_PI * fc / fs);
    double Q = fc / fb;

    double den = 1 + (K / V0 / Q) + K * K;

    b[0] = (1 + K / Q + K * K) / den;
    b[1] = (2 * K * K - 2) / den;
    b[2] = (1 - K / Q + K * K) / den;

    a[0] = 1;
    a[1] = (2 * K * K - 2) / den;
    a[2] = (1 - K / V0 / Q + K * K) / den;
}

/**
 * @brief This function calculates the coefficients for a low frequency boost
 * @param fc The center frequency
 * @param fb The bandwidth
 * @param fs The sampling frequency.
 * @param G  The gain in dB.
 * @param b  A pointer to an array of size 3, where the calculated coefficients for the numerator of the filter will be stored.
 * @param a  A pointer to an array of size 3, where the calculated coefficients for the denominator of the filter will be stored.
 * @return void
 * @note The function low frequency boost? actual is cut, like this ShelvingLF_Boost(500, 500, 16000, -10, b, a);
 */
void ShelvingLF_Boost(double fc, double fb, double fs, double G, double *b, double *a)
{
    double V0 = pow(10, G / 20);
    double K = tan(M_PI * fc / fs);
    double Q = fc / fb;

    double den = 1 + sqrt(2) * K + K * K;

    b[0] = (1 + sqrt(2 * V0) * K + V0 * K * K) / den;
    b[1] = (2 * V0 * K * K - 2) / den;
    b[2] = (1 - sqrt(2 * V0) * K + V0 * K * K) / den;

    a[0] = 1;
    a[1] = (2 * K * K - 2) / den;
    a[2] = (1 - sqrt(2) * K + K * K) / den;
}

/**
 * @brief This function calculates the coefficients for a low frequency cut
 * @param fc The center frequency
 * @param fb The bandwidth
 * @param fs The sampling frequency.
 * @param G  The gain in dB.
 * @param b  A pointer to an array of size 3, where the calculated coefficients for the numerator of the filter will be stored.
 * @param a  A pointer to an array of size 3, where the calculated coefficients for the denominator of the filter will be stored.
 * @return void
 * @note The function low frequency cut? actual is boost, like this ShelvingLF_Cut(500, 500, 16000, 10, b, a);
 */
void ShelvingLF_Cut(double fc, double fb, double fs, double G, double *b, double *a)
{
    double V0 = pow(10, G / 20);
    double K = tan(M_PI * fc / fs);
    double Q = fc / fb;

    double den = V0 + sqrt(2 * V0) * K + K * K;

    b[0] = (V0 + V0 * sqrt(2) * K + V0 * K * K) / den;
    b[1] = (2 * V0 * K * K - 2 * V0) / den;
    b[2] = (V0 - V0 * sqrt(2) * K + V0 * K * K) / den;

    a[0] = 1;
    a[1] = (2 * K * K - 2 * V0) / den;
    a[2] = (V0 - sqrt(2 * V0) * K + K * K) / den;
}

/**
 * @brief This function calculates the coefficients for a high frequency boost
 * @param fc The center frequency
 * @param fb The bandwidth
 * @param fs The sampling frequency.
 * @param G  The gain in dB.
 * @param b  A pointer to an array of size 3, where the calculated coefficients for the numerator of the filter will be stored.
 * @param a  A pointer to an array of size 3, where the calculated coefficients for the denominator of the filter will be stored.
 * @return void
 * @note The function low frequency boost? actual is cut, like this ShelvingHF_Boost(7500, 500, 16000, -10, b, a);
 */
void ShelvingHF_Boost(double fc, double fb, double fs, double G, double *b, double *a)
{
    double V0 = pow(10, G / 20);
    double K = tan(M_PI * fc / fs);
    double Q = fc / fb;

    double den = 1 + sqrt(2) * K + K * K;

    b[0] = (V0 + sqrt(2 * V0) * K + K * K) / den;
    b[1] = (2 * K * K - 2 * V0) / den;
    b[2] = (V0 - sqrt(2 * V0) * K + K * K) / den;

    a[0] = 1;
    a[1] = (2 * K * K - 2) / den;
    a[2] = (1 - sqrt(2) * K + K * K) / den;
}

/**
 * @brief This function calculates the coefficients for a high frequency cut
 * @param fc The center frequency
 * @param fb The bandwidth
 * @param fs The sampling frequency.
 * @param G  The gain in dB.
 * @param b  A pointer to an array of size 3, where the calculated coefficients for the numerator of the filter will be stored.
 * @param a  A pointer to an array of size 3, where the calculated coefficients for the denominator of the filter will be stored.
 * @return void
 * @note The function high frequency cut? actual is boost, like this ShelvingHF_Cut(7500, 500, 16000, 10, b, a);
 */
void ShelvingHF_Cut(double fc, double fb, double fs, double G, double *b, double *a)
{
    double V0 = pow(10, G / 20);
    double K = tan(M_PI * fc / fs);
    double Q = fc / fb;

    double den = 1 + sqrt(2 * V0) * K + V0 * K * K;

    b[0] = (V0 + V0 * sqrt(2) * K + V0 * K * K) / den;
    b[1] = (2 * V0 * K * K - 2 * V0) / den;
    b[2] = (V0 - V0 * sqrt(2) * K + V0 * K * K) / den;

    a[0] = 1;
    a[1] = (2 * V0 * K * K - 2) / den;
    a[2] = (1 - sqrt(2 * V0) * K + V0 * K * K) / den;
}

/**
 * @brief 计算级联型数字滤波器的频率响应、幅频响应和相频响应
 *
 * @param b ：长度为ns*(n+1)，存放滤波器分子的系数b[j][i]，j为阶数i为每阶系数
 * @param a ：长度为ns*(n+1)，存放滤波器分母的系数a[j][i]
 * @param n ：级联型滤波器每节的阶数
 * @param ns ：级联型滤波器的n阶节数L
 * @param x ：长度为len，sign=0时存放滤波器频率响应的实部，
 * sign=1时存放滤波器幅频响应H(w),sign=2时存放分贝表示的滤波器幅频响应H(w)
 * @param y ：长度为len，sign=0时存放滤波器频率响应的虚部，
 * sign=1,2的时候存放滤波器的相频响应
 * @param len ：滤波器响应的长度
 * @param sign ：sign=0时为滤波器频率响应的实部和虚部，sign=1时输出幅频响应和相频响应
 * sign=2时输出dB形式的响应
 */
void Gainc(double *b, double *a, int n, int ns, double *x, double *y, int len, int sign)
{
    int i, j, k, n1;
    double ar, ai, br, bi, zr, zi, im, re, den, numi, numr, freq, temp;
    double hr, hi, tr, ti;

    n1 = n + 1;
    for (k = 0; k < len; k++)
    {
        freq = k * 0.5 / (len - 1);
        zr = cos(-8.0 * atan(1.0) * freq);
        zi = sin(-8.0 * atan(1.0) * freq);
        x[k] = 1.0;
        y[k] = 0.0;
        for (j = 0; j < ns; j++)
        {
            br = 0.0;
            bi = 0.0;
            for (i = n; i > 0; i--)
            {
                re = br;
                im = bi;
                br = (re + b[j * n1 + i]) * zr - im * zi;
                bi = (re + b[j * n1 + i]) * zi + im * zr;
            }
            ar = 0.0;
            ai = 0.0;
            for (i = n; i > 0; i--)
            {
                re = ar;
                im = ai;
                ar = (re + a[j * n1 + i]) * zr - im * zi;
                ai = (re + a[j * n1 + i]) * zi + im * zr;
            }
            br = br + b[j * n1 + 0];
            ar = ar + 1.0;
            numr = ar * br + ai * bi;
            numi = ar * bi - ai * br;
            den = ar * ar + ai * ai;
            hr = numr / den;
            hi = numi / den;
            tr = x[k] * hr - y[k] * hi;
            ti = x[k] * hi + y[k] * hr;
            x[k] = tr;
            y[k] = ti;
        }

        switch (sign)
        {
        case 1:
            temp = sqrt(x[k] * x[k] + y[k] * y[k]);
            if (temp != 0.0)
            {
                y[k] = atan2(y[k], x[k]);
            }
            else
            {
                y[k] = 0.0;
            }
            x[k] = temp;
            break;
        case 2:
            temp = x[k] * x[k] + y[k] * y[k];
            if (temp != 0.0)
            {
                y[k] = atan2(y[k], x[k]);
            }
            else
            {
                temp = 1e-40;
                y[k] = 0.0;
            }

            x[k] = 10.0 * log10(temp); // 20.0*log10(temp)?
            break;
        }
    }
}
```


### main.c

```c
#include "eq_filterdesigner.h"
#include <math.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#if 1
int main(int argc, char **argv)
{
    int i;
    double fs = 48000;
    double fc1[] = {250, 500, 750, 1000, 1500, 2000, 3000, 4000, 6000, 8000, 12000, 16000, 20000};
    double gain1[] = {1, 2, 3, 4, 5, 3, 1, -1, -3, -5, -2, -1, 2};
    double fb_q = 2;
    double fb1[20] = {0};
    for (i = 0; i < 13; i++)
    {
        fb1[i] = fc1[i] / fb_q;
    }
    double bb1[200] = {0};
    double aa1[200] = {0};
    double b[10] = {0};
    double a[10] = {0};
    int valid_filter = 0;

    for (i = 0; i < 13; i++)
    {
        if (fabs(gain1[i]) > 1e-6)
        {
            if (gain1 > 0)
            {
                PeakBoost(fc1[i], fb1[i], fs, gain1[i], b, a);
            }
            else
            {
                PeakCut(fc1[i], fb1[i], fs, gain1[i], b, a);
            }
            // copy data
            memcpy(&bb1[valid_filter * 3], b, sizeof(double) * 3);
            memcpy(&aa1[valid_filter * 3], a, sizeof(double) * 3);
            valid_filter++;
        }
    }

    FILE *fp = NULL;
    fp = fopen("eq_coeffs.txt", "w");
    if (fp == NULL)
    {
        printf("Error opening file!\n");
        return 1;
    }
    for (i = 0; i < valid_filter; i++)
    {
        fprintf(fp, "filter %d:\n", i + 1);
        fprintf(fp, "b = %.10f, %.10f, %.10f\n", bb1[i * 3], bb1[i * 3 + 1], bb1[i * 3 + 2]);
        fprintf(fp, "a = %.10f, %.10f, %.10f\n", aa1[i * 3], aa1[i * 3 + 1], aa1[i * 3 + 2]);
    }
    fclose(fp);

    // calcute the filter coefficients response
    double xx1[24000];
    double yy1[24000];
    memset(xx1, 0, sizeof(xx1));
    memset(yy1, 0, sizeof(yy1));

    Gainc(bb1, aa1, 2, valid_filter, xx1, yy1, 24000, 2);
    fp = NULL;
    fp = fopen("eq_response.csv", "w");
    if (fp == NULL)
    {
        printf("Error opening file!\n");
        return 1;
    }
    for (i = 0; i < 24000; i++)
    {
        fprintf(fp, "%d,%.10f,%.10f\n", i, xx1[i], yy1[i]);
    }
    fclose(fp);

    printf("done!\n");
    return 0;
}
#else
int main(int argc, char **argv)
{
    int i;
    double fs = 48000;
    double fc1[] = {250, 500, 750, 1000, 1500, 2000, 3000, 4000, 6000, 8000, 12000, 16000, 20000};
    double gain1[] = {1, 2, 3, 4, 5, 3, 1, -1, -3, -5, -2, -1, 2};
    double fb_q = 2;
    double fb1[20] = {0};
    for (i = 0; i < 13; i++)
    {
        fb1[i] = fc1[i] / fb_q;
    }
    double bb1[200] = {0};
    double aa1[200] = {0};
    double b[10] = {0};
    double a[10] = {0};
    int valid_filter = 0;

    // low frequency shelving filter
    ShelvingLF_Boost(100, 200, fs, -10, b, a);
    // copy data
    memcpy(&bb1[valid_filter * 3], b, sizeof(double) * 3);
    memcpy(&aa1[valid_filter * 3], a, sizeof(double) * 3);
    valid_filter++;
    // eq filter
    for (i = 0; i < 13; i++)
    {
        if (fabs(gain1[i]) > 1e-6)
        {
            if (gain1 > 0)
            {
                PeakBoost(fc1[i], fb1[i], fs, gain1[i], b, a);
            }
            else
            {
                PeakCut(fc1[i], fb1[i], fs, gain1[i], b, a);
            }
            // copy data
            memcpy(&bb1[valid_filter * 3], b, sizeof(double) * 3);
            memcpy(&aa1[valid_filter * 3], a, sizeof(double) * 3);
            valid_filter++;
        }
    }
    // high frequency shelving filter
    ShelvingHF_Boost(22000, 1000, fs, -10, b, a);
    // copy data
    memcpy(&bb1[valid_filter * 3], b, sizeof(double) * 3);
    memcpy(&aa1[valid_filter * 3], a, sizeof(double) * 3);
    valid_filter++;

    FILE *fp = NULL;
    fp = fopen("eq_coeffs.txt", "w");
    if (fp == NULL)
    {
        printf("Error opening file!\n");
        return 1;
    }
    for (i = 0; i < valid_filter; i++)
    {
        fprintf(fp, "filter %d:\n", i + 1);
        fprintf(fp, "b = %.10f, %.10f, %.10f\n", bb1[i * 3], bb1[i * 3 + 1], bb1[i * 3 + 2]);
        fprintf(fp, "a = %.10f, %.10f, %.10f\n", aa1[i * 3], aa1[i * 3 + 1], aa1[i * 3 + 2]);
    }
    fclose(fp);

    // calcute the filter coefficients response
    double xx1[24000];
    double yy1[24000];
    memset(xx1, 0, sizeof(xx1));
    memset(yy1, 0, sizeof(yy1));

    Gainc(bb1, aa1, 2, valid_filter, xx1, yy1, 24000, 2);
    fp = NULL;
    fp = fopen("eq_response2.csv", "w");
    if (fp == NULL)
    {
        printf("Error opening file!\n");
        return 1;
    }
    for (i = 0; i < 24000; i++)
    {
        fprintf(fp, "%d,%.10f,%.10f\n", i, xx1[i], yy1[i]);
    }
    fclose(fp);

    printf("done!\n");
    return 0;
}
#endif
```