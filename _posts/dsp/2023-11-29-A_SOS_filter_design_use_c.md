---
title: 巴特沃斯滤波器的双二阶方式
author: lixinghui
date: 2023-11-29 10:10:00 +0800
categories: [DSP, filter]
tags: [数字信号处理]
---

这是巴特沃斯滤波器的双二阶(Second-Order Section)生成方式，在多阶滤波器参数无法用单精度来表示的时候，需要使用SOS的方式来生成，这样才能使得每一级的滤波器使用单精度正常运行。

>   在多阶滤波器中，分子b的数据会很小，这里是该数据单精度浮点数不能表示了，并不能通过放大来处理
{: .prompt-tip }

关于滤波器是否转单精度会影响响应可以在matlab中将参数使用`single(b)`来转换为单精度数据，再用`fvtool`查看响应。

[Github的C++工程](https://github.com/ruohoruotsi/Butterworth-Filter-Design)

这个是对该C++工程转换为C语言的，仅对二阶滤波器进行了转换，四阶的EQ没有转换。在《一种IIR滤波器生成方式-C语言》这一篇中，低通和高通已经是二阶了，但是带通带阻是四阶的，这里将带通带阻都转化为二阶了。在单片机使用中可以更换标准C库的内存分配。

代码如下：

## biquad.h

```c
#ifndef BIQUAD_H
#define BIQUAD_H

// A biquad filter expression:
// y[n] = b0 * x[n] + b1 * x[n-1] + b2 * x[n-2] + a1 * y[n-1] + a2 * y[n-2];


typedef struct
{
    double b0, b1, b2, a0, a1, a2;  // second order section variable
    // double a3, a4, b3, b4;      // fourth order section variables
}Biquad;


// Coefficients for a DF2T fourth order section (Used for EQ filters)
void DF2TFourthOrderSection(Biquad *bq,double B0, double B1, double B2, double B3, double B4,
                            double A0, double A1, double A2, double A3, double A4);

// Coefficients for a DF2T biquad section.
void DF2TBiquad(Biquad *bq,double B0, double B1, double B2,
                        double A0, double A1, double A2);

#endif
```

## biquad.c

```c
#include "biquad.h"
#include <stdio.h>
#include <string.h>
#include <math.h>


#if 0
void DF2TFourthOrderSection(Biquad *bq,double B0, double B1, double B2, double B3, double B4,
                            double A0, double A1, double A2, double A3, double A4)
{
    bq->b0 = B0  / A0;
    bq->b1 = B1  / A0;
    bq->b2 = B2  / A0;
    bq->b3 = B3  / A0;
    bq->b4 = B4  / A0;
    
    bq->a1 = (-A1) / A0;  // The negation conforms the Direct-Form II Transposed discrete-time
    bq->a2 = (-A2) / A0;  // filter (DF2T) coefficients to the expectations of the process function.
    bq->a3 = (-A3) / A0;
    bq->a4 = (-A4) / A0;
}
#endif

void DF2TBiquad(Biquad *bq,double B0, double B1, double B2,
                        double A0, double A1, double A2)
{
    bq->b0 = B0  / A0;
    bq->b1 = B1  / A0;
    bq->b2 = B2  / A0;
    bq->a0 = 1.0;
    bq->a1 = A1 / A0;  
    bq->a2 = A2 / A0;  
}
```

## butterworth.h

```c

#ifndef BUTTERWORTH_H
#define BUTTERWORTH_H

#include "biquad.h"
#include <stdbool.h>

typedef struct
{
    double real;
    double imag;
} Complex;

typedef enum
{
    kLoPass = 10000,
    kHiPass = 10001,
    kBandPass = 10002,
    kBandStop = 10003,
    kLoShelf = 10004,
    kHiShelf = 10005,   // high order EQ
    kParametric = 10006 // high order EQ
}FILTER_TYPE;

typedef struct
{
    // Internal state used during computation of coefficients
    double f1, f2;
    int numPoles, numZeros;
    Complex *zeros;
    Complex *poles;

    double Wc; // Omega cutoff == passband edge freq
    double bw; // Bandwidth
    
    double gain;
    double preBLTgain;
    
    int nba;
    double * ba;
}Butterworth;


bool loPass(double fs, double f1, double f2, int filterOrder,
            Biquad *coeffs, double *overallGain);

bool hiPass(double fs, double f1, double f2, int filterOrder,
     Biquad *coeffs, double *overallGain);

bool bandPass(double fs, double f1, double f2, int filterOrder,
              Biquad *coeffs, double *overallGain);

bool bandStop(double fs, double f1, double f2, int filterOrder,
              Biquad *coeffs, double *overallGain);

#endif
```

## butterworth.c

```c
#include "butterworth.h"
#include "biquad.h"
#include <math.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define LOG_OUTPUT 0

#define MAX(a, b) (((a) > (b)) ? (a) : (b))
#define MIN(a, b) (((a) < (b)) ? (a) : (b))

Butterworth bw;

double blt(Complex *sz);
bool s2Z(Butterworth *bw);
bool zp2SOS(Butterworth *bw);
void convert2lopass(Butterworth *bw);
void convert2hipass(Butterworth *bw);
void convert2bandpass(Butterworth *bw);
void convert2bandstop(Butterworth *bw);

void prototypeAnalogLowPass(int filterOrder, Complex *poles)
{
    for (uint32_t k = 0; k < (filterOrder + 1) / 2; k++)
    {
        double theta = (double)(2 * k + 1) * M_PI / (2 * filterOrder);
        double real = -sin(theta);
        double imag = cos(theta);
        poles[2 * k].real = real;
        poles[2 * k].imag = imag;
        poles[2 * k + 1].real = real; // conjugate
        poles[2 * k + 1].imag = -imag;
    }
}

bool coefficients(FILTER_TYPE filter, const double fs, const double freq1_cutoff,
                  const double freq2_cutoff, const int filterOrder,
                  Biquad *coeffs, double *overallGain)
{

    //******************************************************************************
    // Init internal state based on filter design requirements
    bw.zeros = calloc(2 * filterOrder, sizeof(Complex));
    bw.poles = calloc(2 * filterOrder, sizeof(Complex));

    bw.f1 = freq1_cutoff;
    bw.f2 = freq2_cutoff;

    bw.Wc = 0; // Omega cutoff = passband edge freq
    bw.bw = 0;

    //******************************************************************************
    // Prewarp
    bw.f1 = 2 * tan(M_PI * bw.f1 / fs);
    bw.f2 = 2 * tan(M_PI * bw.f2 / fs);

#if LOG_OUTPUT
    printf("\n");
    printf("[Butterworth Filter Design] prewarped f1 = %lf\n", bw.f1);
    printf("[Butterworth Filter Design] prewarped f2 = %lf\n", bw.f2);
#endif

    //******************************************************************************
    // Design basic S-plane poles-only analogue LP prototype

    int nn = filterOrder % 2 ? 1 : 0;
    nn += filterOrder;
    Complex *tempPoles = calloc(nn, sizeof(Complex));
    // Get zeros & poles of prototype analogue low pass.
    prototypeAnalogLowPass(filterOrder, tempPoles);

    // Copy tmppole into poles
    for (int i = 0; i < nn; i++)
    {
        bw.poles[i].real = tempPoles[i].real;
        bw.poles[i].imag = tempPoles[i].imag;
    }
    free(tempPoles);

    bw.numPoles = nn;
    bw.numZeros = 0; // butterworth LP prototype has no zeros.
    bw.gain = 1.0;   // always 1 for the butterworth prototype lowpass.

    //******************************************************************************
    // Convert prototype to target filter type (LP/HP/BP/BS) - S-plane

    // Re-orient BP/BS corner frequencies if necessary
    if (bw.f1 > bw.f2)
    {
        double temp = bw.f2;
        bw.f2 = bw.f1;
        bw.f1 = temp;
    }

    // Cutoff Wc = f2
    switch (filter)
    {
    case kLoPass:
        convert2lopass(&bw);
        break;

    case kHiPass:
        convert2hipass(&bw);
        break;

    case kBandPass:
        convert2bandpass(&bw);
        break;

    case kBandStop:
        convert2bandstop(&bw);
        break;

    default:
    {
#if LOG_OUTPUT
        printf("[Butterworth Filter Design] Unknown Filter Type\n");
#endif
        return false;
    }
    }

    //******************************************************************************
    // SANITY CHECK: Ensure poles are in the left half of the S-plane
    for (uint32_t i = 0; i < bw.numPoles; i++)
    {
        if (bw.poles[i].real > 0)
        {
#if LOG_OUTPUT
            printf("[Butterworth Filter Design] Error: poles must be in the left half plane\n");
#endif
            return false;
        }
    }

    //******************************************************************************
    // Map zeros & poles from S-plane to Z-plane

    bw.nba = 0;
    bw.ba = calloc(2 * MAX(bw.numPoles, bw.numZeros) + 5, sizeof(double));
    bw.preBLTgain = bw.gain;

    if (!s2Z(&bw))
    {
#if LOG_OUTPUT
        printf("[Butterworth Filter Design] Error: s2Z failed\n");
#endif
        return false;
    }

    //******************************************************************************
    // Split up Z-plane poles and zeros into SOS

    if (!zp2SOS(&bw))
    {
#if LOG_OUTPUT
        printf("[Butterworth Filter Design] Error: zp2SOS failed\n");
#endif
        return false;
    }

    // correct the overall gain
    if (filter == kLoPass || filter == kBandPass)
    {                                                         // pre-blt is okay for S-plane
        bw.ba[0] = bw.preBLTgain * (bw.preBLTgain / bw.gain); // 2nd term is how much BLT boosts,
    }
    else if (filter == kHiPass || kBandStop)
    { // HF gain != DC gain
        bw.ba[0] = 1 / bw.ba[0];
    }

    //******************************************************************************
    // Init biquad chain with coefficients from SOS

    *overallGain = bw.ba[0];
    int numFilters = filterOrder / 2;
    if (filter == kBandPass || filter == kBandStop)
    {
        numFilters = filterOrder; // we have double the # of biquad sections
        // IOHAVOC filterOrder is never used again? figure this out FIXME
        // filterOrder *= 2;
    }

    for (uint32_t i = 0; i < numFilters; i++)
    {
        DF2TBiquad(&coeffs[i],
                   1.0,               // b0
                   bw.ba[4 * i + 1],  // b1
                   bw.ba[4 * i + 2],  // b2
                   1.0,               // a0
                   bw.ba[4 * i + 3],  // a1
                   bw.ba[4 * i + 4]); // a2
    }

#if LOG_OUTPUT

    // ba[0] contains your gain factor
    printf("\n");
    printf("[Butterworth Filter Design] preBLTgain = %lf\n", bw.preBLTgain);
    printf("[Butterworth Filter Design] gain = %lf\n", bw.gain);

    printf("[Butterworth Filter Design] ba[0] = %lf\n", bw.ba[0]);
    printf("[Butterworth Filter Design] coeff size = %d\n\n", bw.nba);

    for (uint32_t i = 0; i < (bw.nba - 1) / 4; i++)
    {
        // b0,b1,b2: a0,a1,a2:= 1.0, ba[4*i+1], ba[4*i+2], 1.0, ba[4*i+3], ba[4*i+4]
        printf("[Butterworth Filter Design] Biquads:= 1.0 %lf %lf %lf %lf\n",
               bw.ba[4 * i + 1], bw.ba[4 * i + 2], bw.ba[4 * i + 3], bw.ba[4 * i + 4]);
    }
#endif

    free(bw.ba);
    free(bw.poles);
    free(bw.zeros);
    return true;
}

bool loPass(double fs, double f1, double f2, int filterOrder,
            Biquad *coeffs, double *overallGain)
{
    return coefficients(kLoPass, fs, f1, f2, filterOrder, coeffs, overallGain);
}

bool hiPass(double fs, double f1, double f2, int filterOrder,
            Biquad *coeffs, double *overallGain)
{
    return coefficients(kHiPass, fs, f1, f2, filterOrder, coeffs, overallGain);
}

bool bandPass(double fs, double f1, double f2, int filterOrder,
              Biquad *coeffs, double *overallGain)
{
    return coefficients(kBandPass, fs, f1, f2, filterOrder, coeffs, overallGain);
}

bool bandStop(double fs, double f1, double f2, int filterOrder,
              Biquad *coeffs, double *overallGain)
{
    return coefficients(kBandStop, fs, f1, f2, filterOrder, coeffs, overallGain);
}

// 复数乘法
Complex complexMultiply(Complex *a, Complex *b)
{
    Complex result;
    result.real = a->real * b->real - a->imag * b->imag;
    result.imag = a->real * b->imag + a->imag * b->real;
    return result;
}

// 复数除法
Complex complexDivide(Complex *a, Complex *b)
{
    Complex result;
    double denominator = b->real * b->real + b->imag * b->imag;

    if (denominator == 0.0)
    {
        // fprintf(stderr, "Error: Division by zero.\n");
        result.real = 0.0;
        result.imag = 0.0;
    }
    else
    {
        result.real = (a->real * b->real + a->imag * b->imag) / denominator;
        result.imag = (a->imag * b->real - a->real * b->imag) / denominator;
    }

    return result;
}

// 计算复数的平方根
Complex complexSqrt(const Complex *num)
{
    double r = sqrt(num->real * num->real + num->imag * num->imag);
    double theta = atan2(num->imag, num->real);
    double sqrtR = sqrt(r);

    Complex result;
    result.real = sqrtR * cos(theta / 2.0);
    result.imag = sqrtR * sin(theta / 2.0);

    return result;
}

//******************************************************************************
//
// Z = (2 + S) / (2 - S) is the S-plane to Z-plane bilinear transform
//
// Reference: http://en.wikipedia.org/wiki/Bilinear_transform
//
//******************************************************************************

double blt(Complex *sz)
{

    Complex two = {2.0, 0};
    Complex s = *sz; // sz is an input(S-plane) & output(Z-plane) arg

    Complex tmp1 = {two.real + s.real, two.imag + s.imag};
    Complex tmp2 = {two.real - s.real, two.imag - s.imag};
    *sz = complexDivide(&tmp1, &tmp2);

    // return the gain
    double tmp = sqrt(tmp2.real * tmp2.real + tmp2.imag * tmp2.imag);
    return tmp;
}

//******************************************************************************
//
// Convert poles & zeros from S-plane to Z-plane via Bilinear Tranform (BLT)
//
//******************************************************************************

bool s2Z(Butterworth *bw)
{

    // blt zeros
    for (uint32_t i = 0; i < bw->numZeros; i++)
    {
        bw->gain /= blt(&bw->zeros[i]);
    }

    // blt poles
    for (uint32_t i = 0; i < bw->numPoles; i++)
    {
        bw->gain *= blt(&bw->poles[i]);
    }

    return true;
}

//******************************************************************************
//
// Convert filter poles and zeros to second-order sections
//
// Reference: http://www.mathworks.com/help/signal/ref/zp2sos.html
//
//******************************************************************************

bool zp2SOS(Butterworth *bw)
{
    int filterOrder = MAX(bw->numZeros, bw->numPoles);
    Complex *zerosTempVec = calloc(filterOrder, sizeof(Complex));
    Complex *polesTempVec = calloc(filterOrder, sizeof(Complex));

    // Copy
    for (uint32_t i = 0; i < bw->numZeros; i++)
    {
        memcpy(&zerosTempVec[i], &bw->zeros[i], sizeof(Complex));
    }

    // Add zeros at -1, so if S-plane degenerate case where
    // numZeros = 0 will map to -1 in Z-plane.
    for (uint32_t i = bw->numZeros; i < filterOrder; i++)
    {
        zerosTempVec[i].real = -1;
        zerosTempVec[i].imag = 0;
    }

    // Copy
    for (uint32_t i = 0; i < bw->numPoles; i++)
    {
        memcpy(&polesTempVec[i], &bw->poles[i], sizeof(Complex));
    }

    bw->ba[0] = bw->gain; // store gain

    int numSOS = 0;
    Complex tmp;
    for (uint32_t i = 0; i + 1 < filterOrder; i += 2, numSOS++)
    {
        bw->ba[4 * numSOS + 1] = -(zerosTempVec[i].real + zerosTempVec[i + 1].real);
        tmp = complexMultiply(&zerosTempVec[i], &zerosTempVec[i + 1]);
        bw->ba[4 * numSOS + 2] = tmp.real;
        bw->ba[4 * numSOS + 3] = -(polesTempVec[i].real + polesTempVec[i + 1].real);
        tmp = complexMultiply(&polesTempVec[i], &polesTempVec[i + 1]);
        bw->ba[4 * numSOS + 4] = tmp.real;
    }

    // Odd filter order thus one pair of poles/zeros remains
    if (filterOrder % 2 == 1)
    {
        bw->ba[4 * numSOS + 1] = -zerosTempVec[filterOrder - 1].real;
        bw->ba[4 * numSOS + 2] = 0;
        bw->ba[4 * numSOS + 3] = -polesTempVec[filterOrder - 1].real;
        bw->ba[4 * numSOS + 4] = 0;
        numSOS++;
    }

    // Set output param
    bw->nba = 1 + 4 * numSOS;

    free(zerosTempVec);
    free(polesTempVec);
    return true;
}

//******************************************************************************
// Convert analog lowpass prototype poles to lowpass
//******************************************************************************
void convert2lopass(Butterworth *bw)
{
    bw->Wc = bw->f2; // critical frequency

    bw->gain *= pow(bw->Wc, bw->numPoles);

    bw->numZeros = 0; // poles only
    for (uint32_t i = 0; i < bw->numPoles; i++)
    { // scale poles by the cutoff Wc
        bw->poles[i].real = bw->Wc * bw->poles[i].real;
        bw->poles[i].imag = bw->Wc * bw->poles[i].imag;
    }
}

//******************************************************************************
// Convert lowpass poles & zeros to highpass
// with Wc = f2, use:  hp_S = Wc / lp_S;
//******************************************************************************
void convert2hipass(Butterworth *bw)
{
    bw->Wc = bw->f2; // Critical frequency

    // Calculate gain
    Complex prodz = {1.0, 0.0};
    Complex prodp = {1.0, 0.0};

    for (uint32_t i = 0; i < bw->numZeros; i++)
    {
        Complex temp = {0 - bw->zeros[i].real, 0 - bw->zeros[i].imag};
        prodz = complexMultiply(&prodz, &temp);
    }

    for (uint32_t i = 0; i < bw->numPoles; i++)
    {
        Complex temp = {0 - bw->poles[i].real, 0 - bw->poles[i].imag};
        prodp = complexMultiply(&prodp, &temp);
    }

    bw->gain *= prodz.real / prodp.real;

    int ii = 0;
    // Convert LP poles to HP
    for (uint32_t i = 0; i < bw->numPoles; i++)
    {
        double amp = sqrt(bw->poles[i].real * bw->poles[i].real + bw->poles[i].imag * bw->poles[i].imag);
        if (amp)
        {
            Complex temp = {bw->Wc, 0};
            bw->poles[ii] = complexDivide(&temp, &bw->poles[i]); //  hp_S = Wc / lp_S;
            ii++;
        }
    }

    // Init with zeros, no non-zero values to convert
    bw->numZeros = bw->numPoles;
    for (uint32_t i = 0; i < bw->numZeros; i++)
    {
        memset(&bw->zeros[i], 0, sizeof(Complex));
    }
}

//******************************************************************************
// Convert lowpass poles to bandpass
// use:  bp_S = 0.5 * lp_S * BW +
//				   0.5 * sqrt ( BW^2 * lp_S^2 - 4*Wc^2 )
// where   BW = W2 - W1
//		    Wc^2 = W2 * W1
//******************************************************************************

void convert2bandpass(Butterworth *bw)
{
    bw->bw = bw->f2 - bw->f1;
    bw->Wc = sqrt(bw->f1 * bw->f2);

    // Calculate bandpass gain
    bw->gain *= pow(bw->bw, bw->numPoles - bw->numZeros);

    // Convert LP poles to BP: these two sets of for-loops result in an ordered
    // list of poles and their complex conjugates
    Complex *tempPoles = calloc(2 * bw->numPoles, sizeof(Complex));

    int ii = 0;

    for (uint32_t i = 0; i < bw->numPoles; i++)
    { // First set of poles + conjugates
        double amp = sqrt(bw->poles[i].real * bw->poles[i].real + bw->poles[i].imag * bw->poles[i].imag);
        if (amp)
        {
            double real = 0.5 * bw->poles[i].real * bw->bw;
            double imag = 0.5 * bw->poles[i].imag * bw->bw;
            Complex firstterm = {real, imag};
            Complex temp = complexMultiply(&bw->poles[i], &bw->poles[i]);
            temp.real = (bw->bw * bw->bw) * temp.real - 4 * bw->Wc * bw->Wc;
            temp.imag = (bw->bw * bw->bw) * temp.imag;
            Complex secondterm = complexSqrt(&temp);
            secondterm.real *= 0.5;
            secondterm.imag *= 0.5;
            tempPoles[ii].real = firstterm.real + secondterm.real;
            tempPoles[ii].imag = firstterm.imag + secondterm.imag;
            ii++;
        }
    }

    for (uint32_t i = 0; i < bw->numPoles; i++)
    { // Second set of poles + conjugates
        double amp = sqrt(bw->poles[i].real * bw->poles[i].real + bw->poles[i].imag * bw->poles[i].imag);
        if (amp)
        {
            double real = 0.5 * bw->poles[i].real * bw->bw;
            double imag = 0.5 * bw->poles[i].imag * bw->bw;
            Complex firstterm = {real, imag};
            Complex temp = complexMultiply(&bw->poles[i], &bw->poles[i]);
            temp.real = (bw->bw * bw->bw) * temp.real - 4 * bw->Wc * bw->Wc;
            temp.imag = (bw->bw * bw->bw) * temp.imag;
            Complex secondterm = complexSqrt(&temp);
            secondterm.real *= 0.5;
            secondterm.imag *= 0.5;
            tempPoles[ii].real = firstterm.real - secondterm.real;
            tempPoles[ii].imag = firstterm.imag - secondterm.imag; // complex conjugate
            ii++;
        }
    }

    // Init zeros, no non-zero values to convert
    bw->numZeros = bw->numPoles;
    for (uint32_t i = 0; i < bw->numZeros; i++)
    {
        bw->zeros[i].real = 0.0;
        bw->zeros[i].imag = 0.0;
    }

    // Copy converted poles to output array
    for (uint32_t i = 0; i < ii; i++)
    {
        memcpy(&bw->poles[i], &tempPoles[i], sizeof(Complex));
    }

    bw->numPoles = ii;
    free(tempPoles);
}

//******************************************************************************
// Convert lowpass poles to bandstop
// use:  bs_S = 0.5 * BW / lp_S +
//				   0.5 * sqrt ( BW^2 / lp_S^2 - 4*Wc^2 )
// where   BW = W2 - W1
//		 Wc^2 = W2 * W1
//******************************************************************************

void convert2bandstop(Butterworth *bw)
{
    bw->bw = bw->f2 - bw->f1;
    bw->Wc = sqrt(bw->f1 * bw->f2);

    // Compute gain
    Complex prodz = {1.0, 0.0};
    Complex prodp = {1.0, 0.0};

    for (uint32_t i = 0; i < bw->numZeros; i++)
    {
        Complex temp = {0 - bw->zeros[i].real, 0 - bw->zeros[i].imag};
        prodz = complexMultiply(&prodz, &temp);
    }

    for (uint32_t i = 0; i < bw->numPoles; i++)
    {
        Complex temp = {0 - bw->poles[i].real, 0 - bw->poles[i].imag};
        prodp = complexMultiply(&prodp, &temp);
    }

    bw->gain *= prodz.real / prodp.real;

    // Convert LP zeros to band stop
    bw->numZeros = bw->numPoles;
    Complex *ztmp = calloc(2 * bw->numZeros, sizeof(Complex));
    for (uint32_t i = 0; i < bw->numZeros; i++)
    {
        ztmp[2 * i].real = 0.0;
        ztmp[2 * i].imag = bw->Wc;
        ztmp[2 * i + 1].real = 0.0; // complex conjugate
        ztmp[2 * i + 1].imag = 0 - bw->Wc;
    }

    Complex *tempPoles = calloc(2 * bw->numPoles, sizeof(Complex));
    int ii = 0;
    for (uint32_t i = 0; i < bw->numPoles; i++)
    { // First set of poles + conjugates
        double amp = sqrt(bw->poles[i].real * bw->poles[i].real + bw->poles[i].imag * bw->poles[i].imag);
        if (amp)
        {
            Complex term1 = {0.5 * bw->bw, 0};
            Complex term11 = complexDivide(&term1, &bw->poles[i]);

            Complex term2 = {bw->bw * bw->bw, 0};
            Complex temp = complexMultiply(&bw->poles[i], &bw->poles[i]);
            Complex temp2 = complexDivide(&term2, &temp);
            temp2.real -= (4 * bw->Wc * bw->Wc);
            Complex term22 = complexSqrt(&temp2);
            term22.real *= 0.5;
            term22.imag *= 0.5;

            tempPoles[ii].real = term11.real + term22.real;
            tempPoles[ii].imag = term11.imag + term22.imag;
            ii++;
        }
    }

    for (uint32_t i = 0; i < bw->numPoles; i++)
    { // First set of poles + conjugates
        double amp = sqrt(bw->poles[i].real * bw->poles[i].real + bw->poles[i].imag * bw->poles[i].imag);
        if (amp)
        {
            Complex term1 = {0.5 * bw->bw, 0};
            Complex term11 = complexDivide(&term1, &bw->poles[i]);

            Complex term2 = {bw->bw * bw->bw, 0};
            Complex temp = complexMultiply(&bw->poles[i], &bw->poles[i]);
            Complex temp2 = complexDivide(&term2, &temp);
            temp2.real -= (4 * bw->Wc * bw->Wc);
            Complex term22 = complexSqrt(&temp2);
            term22.real *= 0.5;
            term22.imag *= 0.5;

            tempPoles[ii].real = term11.real - term22.real;
            tempPoles[ii].imag = term11.imag - term22.imag;
            ii++;
        }
    }

    // Copy converted zeros to output array
    for (int i = 0; i < 2 * bw->numZeros; i++)
    {
        memcpy(&bw->zeros[i], &ztmp[i], sizeof(Complex));
    }
    bw->numZeros = 2 * bw->numZeros;

    // Copy converted poles to output array
    for (int i = 0; i < ii; i++)
    {
        memcpy(&bw->poles[i], &tempPoles[i], sizeof(Complex));
    }
    bw->numPoles = ii;

    free(ztmp);
    free(tempPoles);
}
```

## main.c

```c
#include "butterworth.h"
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char **argv)
{
    double fs = 1000;
    double f1 = 0;
    double f2 = 200;
    double filter_order = 4;
    Biquad *coeff = NULL;
    double overallGain = 1.0;

    FILE *fp = fopen("1.txt", "w");

    /*需要对coeff参数进行内存释放*/
    /*低通/高通，coeff为filter_order/2，带通/带阻为filter_order*/
    /*低通时截止频率为f1，f2=0；高通时截止频率为f2，f1=0；*/
    coeff = calloc(filter_order/2, sizeof(Biquad));
    hiPass(fs, f1, f2, filter_order, coeff, &overallGain);

    fprintf(fp, "g=%.12lf;\n", overallGain);
    for (uint32_t i = 0; i < filter_order/2; i++)
    {
        // fprintf(fp, "[Butterworth Filter Design] Biquads:\n");
        fprintf(fp, "b%d= [%.12lf %.12lf %.12lf];\n", i, coeff[i].b0, coeff[i].b1, coeff[i].b2);
        fprintf(fp, "a%d= [%.12lf %.12lf %.12lf];\n", i, coeff[i].a0, coeff[i].a1, coeff[i].a2);
    }
    fprintf(fp, "b=conv(b0,b1).*g;\na=conv(a0,a1);\n");

    free(coeff);
    fclose(fp);
    printf("done\n");
    return 0;
}
```

