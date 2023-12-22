---
title: 一些快速计算数学函数
author: lixinghui
date: 2023-7-10 14:10:00 +0800
categories: [DSP, fast_math]
tags: [数字信号处理]
math: true
render_with_liquid: false
---

前提提要：

这些函数都是用于单片机运算中的，在电脑端的运行，默认数学库的的函数应该会比这些函数更快。

当然不同单片机运行这些函数的速度不同，这里仅以`STM32F103ZET6`芯片来做了测试。



函数如下：

## `fast_math.h`

```c
#ifndef _FAST_MATH_H
#define _FAST_MATH_H

/***************log function*****************/
float fast_log2(float val);
float fast_log(float x);

float fastlog2(float x);
float fastlog(float x);

float fasterlog2(float x);
float fasterlog(float x);

/*******************pow2 function****************/
float fastpow2(float p);
float fasterpow2(float p);

/*****************pow function*******************/
float fastpow(float x, float p);
float fasterpow(float x, float p);

/*******************exp function****************/
float fastexp(float p);
float fasterexp(float p);

/*****************sin function*******************/
float fastsinfull (float x);
float fastersinfull (float x);

/*****************cos function*******************/
float fastcosfull (float x);
float fastercosfull (float x);

/*****************cos function*******************/
float fasttanfull (float x);
float fastertanfull (float x);

/*****************sqrt function*******************/
float fast_sqrt(float x);


#endif

```



## `fast_math.c`

```c
#include "fast_math.h"
#include <stdint.h>

/*
 * testing environment:STM32F103ZET6 No float point unit
 * math lib log2f function running 10000 time spend time 187.068604 ms
 * fast lib fast_log2 function running 10000 time spend time 74.018150 ms
 * fast lib fastlog2 function running 10000 time spend time 125.626305 ms
 * fast lib fasterlog2 function running 10000 time spend time 39.420555 ms
 * for example the function log2
 * fasterlog2():The calculation is the fastest, and the error is also the largest
 * fast_log2():Second to speed and second to error
 * fastlog2():The calculation is slow, but the error is also minimal.
 */

/**************************************************************************************/
/*
 * @the error value in:-0.0049~+0.0049
 * the answer is periodic
 */
float fast_log2(float val)
{
    int *const exp_ptr = (int *)(&val);
    int x = *exp_ptr;
    const int log_2 = ((x >> 23) & 0xff) - 0x80;
    x &= ~(0xff << 23);
    x += 0x7f << 23;
    *exp_ptr = x;

    // val = ((-1.0f / 3) * val + 2) * val - 2.0f / 3; // (1)
    val = (-0.34484843f * val + 2.02466578f) * val - 0.67487759f; // (1)

    return (val + log_2);
}
/*
 * Use the bottom-changing formula:
 * log(x)=log2(x)/log2(e)
 * =(1/log2(e))*log2(x)
 * =0.693147180559945 * log2(x)
 */
float fast_log(float x)
{
    return 0.69314718f * fast_log2(x);
}

/**************************************************************************************/
/*
 * @the error value in:-3.8147e-06~+1.4305e-04
 * the answer is periodic
 */
float fastlog2(float x)
{
    union
    {
        float f;
        uint32_t i;
    } vx = {x};
    union
    {
        uint32_t i;
        float f;
    } mx = {(vx.i & 0x007FFFFF) | 0x3f000000};
    float y = vx.i;
    y *= 1.1920928955078125e-7f;

    return y - 124.22551499f - 1.498030302f * mx.f - 1.72587999f / (0.3520887068f + mx.f);
}
float fastlog(float x)
{
    return 0.69314718f * fastlog2(x);
}

/**************************************************************************************/
/*
 * @the error value in:-0.0573~+0.0288
 * the answer is periodic
 */
float fasterlog2(float x)
{
    union
    {
        float f;
        uint32_t i;
    } vx = {x};
    float y = vx.i;
    y *= 1.1920928955078125e-7f;
    return y - 126.94269504f;
}

float fasterlog(float x)
{
    //  return 0.69314718f * fasterlog2 (x);
    union
    {
        float f;
        uint32_t i;
    } vx = {x};
    float y = vx.i;
    y *= 8.2629582881927490e-8f;
    return y - 87.989971088f;
}

/*
 * math lib pow2 function running 10000 time spend time 507.742706 ms
 * fast lib fastpow2 function running 10000 time spend time 145.764008 ms
 * fast lib fasterpow2 function running 10000 time spend time 30.519209 ms
 */

/**************************************************************************************/
/*
 * @the error percent in:-0.007%~+0.002%
 * the answer is periodic
 */
float fastpow2(float p)
{
    float offset = (p < 0) ? 1.0f : 0.0f;
    float clipp = (p < -126) ? -126.0f : p;
    int w = clipp;
    float z = clipp - w + offset;
    union
    {
        uint32_t i;
        float f;
    } v = {(uint32_t)((1 << 23) * (clipp + 121.2740575f + 27.7280233f / (4.84252568f - z) - 1.49012907f * z))};

    return v.f;
}

/**************************************************************************************/
/*
 * @the error percent in:-3.9%~+2%
 * the answer is periodic
 */
float fasterpow2(float p)
{
    float clipp = (p < -126) ? -126.0f : p;
    union
    {
        uint32_t i;
        float f;
    } v = {(uint32_t)((1 << 23) * (clipp + 126.94269504f))};
    return v.f;
}

/*
 * math lib powf function running 10000 time spend time 742.190063 ms
 * fast lib fastpow function running 10000 time spend time 282.480072 ms
 * fast lib fasterpow function running 10000 time spend time 79.194290 ms
 */
/**************************************************************************************/
/*
 * Calculation formula:x^y = (2^log2(x))^y
 * = 2^(log2(x)*y)
 * As the calculation result increases, the error also increases
 * summary:the error is small
 */
float fastpow(float x, float p)
{
    return fastpow2(p * fastlog2(x));
}

/**************************************************************************************/
/*
 * the error percent is large , the function can't use ×
 * ××××××××××××××××××××××××××
 */
float fasterpow(float x, float p)
{
    return fasterpow2(p * fasterlog2(x));
}

/*
 * math lib expf function running 10000 time spend time 189.627182 ms
 * fast lib fastexp function running 10000 time spend time 165.667053 ms
 * fast lib fasterexp function running 10000 time spend time 50.151917 ms
 */
/**************************************************************************************/
/*
 * Use the bottom-changing formula:
 * e=2^(log2(e))
 * e^x = exp(x) = (2^(log2(e)))^x
 * = 2^( log2(e)*x )
 * = 2^(1.442695040888963 * x)
 */
/*
 * the error percent in:-0.007043% ~ +0.001872%
 * the answer is periodic
 */
float fastexp(float p)
{
    return fastpow2(1.442695040f * p);
}

/**************************************************************************************/
/*
 * the error percent in:-3.892146% ~ +2.014498%
 * the answer is periodic
 */
float fasterexp(float p)
{
    return fasterpow2(1.442695040f * p);
}


/*
 * math lib sinf function running 10000 time spend time 205.314011 ms
 * fast lib fastsinfull function running 10000 time spend time 190.587860 ms
 * fast lib fastersinfull function running 10000 time spend time 143.611755 ms
 */
/**************************************************************************************/
/*
 * The range of input x is supported between -PI ~ +PI
 * The range of error in:-3.892928300000542e-05 ~ +3.891438300000771e-05
 */
static float fastsin(float x)
{
    static const float fouroverpi = 1.2732395447351627f;
    static const float fouroverpisq = 0.40528473456935109f;
    static const float q = 0.78444488374548933f;
    union
    {
        float f;
        uint32_t i;
    } p = {0.20363937680730309f};
    union
    {
        float f;
        uint32_t i;
    } r = {0.015124940802184233f};
    union
    {
        float f;
        uint32_t i;
    } s = {-0.0032225901625579573f};

    union
    {
        float f;
        uint32_t i;
    } vx = {x};
    uint32_t sign = vx.i & 0x80000000;
    vx.i = vx.i & 0x7FFFFFFF;

    float qpprox = fouroverpi * x - fouroverpisq * x * vx.f;
    float qpproxsq = qpprox * qpprox;

    p.i |= sign;
    r.i |= sign;
    s.i ^= sign;

    return q * qpprox + qpproxsq * (p.f + qpproxsq * (r.f + qpproxsq * s.f));
}

/**************************************************************************************/
/*
 * The range of input x is supported between -INF ~ +INF
 * The range of error in:-3.901869099999511e-05 ~ +3.910809700000129e-05
 */
float fastsinfull(float x)
{
    static const float twopi = 6.2831853071795865f;
    static const float invtwopi = 0.15915494309189534f;

    int k = x * invtwopi;
    float half = (x < 0) ? -0.5f : 0.5f;
    return fastsin((half + k) * twopi - x);
}

/**************************************************************************************/
/*
 * The range of input x is supported between -PI ~ +PI
 * The range of error in:-8.890628809999912e-04 ~ +8.890330790000123e-04
 */
static float fastersin(float x)
{
    static const float fouroverpi = 1.2732395447351627f;
    static const float fouroverpisq = 0.40528473456935109f;
    static const float q = 0.77633023248007499f;
    union
    {
        float f;
        uint32_t i;
    } p = {0.22308510060189463f};

    union
    {
        float f;
        uint32_t i;
    } vx = {x};
    uint32_t sign = vx.i & 0x80000000;
    vx.i &= 0x7FFFFFFF;

    float qpprox = fouroverpi * x - fouroverpisq * x * vx.f;

    p.i |= sign;

    return qpprox * (q + p.f * qpprox);
}

/**************************************************************************************/
/*
 * The range of input x is supported between -INF ~ +INF
 * The range of error in:-8.890926840000035e-04 ~ +8.891969919999909e-04
 */
float fastersinfull(float x)
{
    static const float twopi = 6.2831853071795865f;
    static const float invtwopi = 0.15915494309189534f;

    int k = x * invtwopi;
    float half = (x < 0) ? -0.5f : 0.5f;
    return fastersin((half + k) * twopi - x);
}

/**************************************************************************************/
/*
 * The range of input x is supported between -PI ~ +PI
 * The range of error in:-3.904104200000424e-05 ~ +3.904104300000988e-05
 */
static float fastcos (float x)
{
  static const float halfpi = 1.5707963267948966f;
  static const float halfpiminustwopi = -4.7123889803846899f;
  float offset = (x > halfpi) ? halfpiminustwopi : halfpi;
  return fastsin (x + offset);
}

/**************************************************************************************/
/*
 * The range of input x is supported between -PI ~ +PI
 * The range of error in:-0.006543755532000 ~ +0.006543755532000
 */
static float fastercos (float x)
{
  static const float twooverpi = 0.63661977236758134f;
  static const float p = 0.54641335845679634f;

  union { float f; uint32_t i; } vx = { x };
  vx.i &= 0x7FFFFFFF;

  float qpprox = 1.0f - twooverpi * vx.f;

  return qpprox + p * qpprox * (1.0f - qpprox * qpprox);
}

/**************************************************************************************/
/*
 * The range of input x is supported between -INF ~ +INF
 * The range of error in:-3.898143699999912e-05 ~ +3.878772300000555e-05
 */
float fastcosfull (float x)
{
  static const float halfpi = 1.5707963267948966f;
  return fastsinfull (x + halfpi);
}

/**************************************************************************************/
/*
 * The range of input x is supported between -INF ~ +INF
 * The range of error in:-8.891224860000102e-04 ~ +8.888840680000010e-04
 */
float fastercosfull (float x)
{
  static const float halfpi = 1.5707963267948966f;
  return fastersinfull (x + halfpi);
}

/**************************************************************************************/
float fasttanfull (float x)
{
  static const float twopi = 6.2831853071795865f;
  static const float invtwopi = 0.15915494309189534f;

  int k = x * invtwopi;
  float half = (x < 0) ? -0.5f : 0.5f;
  float xnew = x - (half + k) * twopi;

  return fastsin (xnew) / fastcos (xnew);
}

/**************************************************************************************/
float fastertanfull (float x)
{
  static const float twopi = 6.2831853071795865f;
  static const float invtwopi = 0.15915494309189534f;

  int k = x * invtwopi;
  float half = (x < 0) ? -0.5f : 0.5f;
  float xnew = x - (half + k) * twopi;

  return fastersin (xnew) / fastercos (xnew);
}


/*
 * math lib sqrtf function running 10000 time spend time 93.262726 ms
 * fast lib fast_sqrt function running 10000 time spend time 146.496857 ms
 * the fast_sqrt is slow than math lib sqrtf function!!!!!!!
 */
/**************************************************************************************/
/*
 * the range of error percent in: -0.000013% ~ +0.000471%
 */
float fast_sqrt(float x)
{
    float xhalf = 0.5f * x;
    int i = *(int *)&x;
    i = 0x5f3759df - (i >> 1); // Compute the first approximate root
    x = *(float *)&i;
    x = x * (1.5f - xhalf * x * x); // Newton iteration method
    x = x * (1.5f - xhalf * x * x);
    // x = x * (1.5f - xhalf * x * x);
    return (1 / x);
}

```

