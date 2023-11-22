---
title: 短时傅里叶变换
author: lixinghui
date: 2023-3-7 14:10:00 +0800
categories: [DSP, FFT]
tags: [数字信号处理]
math: true
render_with_liquid: false
---

## 离散傅里叶变换公式：

$$
X(k)=\sum_{n=0}^{N-1}x(n)e^{-i\frac{2\pi}{N}nk},(k=0,1,2,...,N-1) 
$$

$$
x(n)=\frac{1}{N}\sum_{k=0}^{N-1}X(k)e^{-i\frac{2\pi}{N}nk},(n=0,1,2,...,N-1)
$$

其中输入信号x(n)为时域信号，经过傅里叶变换之后的信号X(k)为频域信号。

在实际使用中一般使用速度更快的快速傅里叶变换(FFT)来进行计算，对时域的信号取N点{N一般取$2^n$}，复数FFT计算出来的频域信号一般为N个复数点(2N个数值)，其中关于$\frac{N}{2}+1$点做奇数对称(实数相等，虚数相反)，幅值相同，相位相反，所以一般计算只取前$\frac{N}{2}+1$点做计算。

每个点的频率输出为：第一个点为0Hz，步进为$\frac{fs}{N}$，最大值为$\frac{N-1}{N}fs$，根据奈奎斯特采样定律，最大频率为$\frac{1}{2}fs$，根据前面所说只取前$\frac{N}{2}+1$点做计算，则频率取值范围为0~$\frac{1}{2}fs$。

根据需求可以对频域信号做增益，第一个点为0Hz信号，对低频声音进行衰减可以降低噪声分量，原因为这些频段的人声较少，噪声很多，且能量占比较大。

傅里叶变换后的分辨率为：$\frac{fs}{N}$，如16K的采样率，做128点的傅里叶变换，则频域的分辨率为$16*1000/128=125Hz$.



## 提高分辨率：

部分时候输入序列长度不够，导致频域的精度不够，无法精确的定位频域的位置，这时就需要提高频域的分辨率，可以在时域序列后面进行补零操作，做点数更多的傅里叶变换。

如8点的序列：
```
x=[29	4	3	27	39	47	7	29]

fft(x)=
000000000000 + 0.00000000000000i	
-38.9913780286485 + 35.8198051533946i	
58.0000000000000 + 5.00000000000000i	
18.9913780286485 + 27.8198051533946i	
-29.0000000000000 + 0.00000000000000i	
18.9913780286485 - 27.8198051533946i	
58.0000000000000 - 5.00000000000000i	
-38.9913780286485 - 35.8198051533946i

x2=[29	4	3	27	39	47	7	29	0	0	0	0	0	0	0	0]
fft(x2)=
185.000000000000 + 0.00000000000000i	
-4.57908408483015 - 127.066706485749i	
-38.9913780286485 + 35.8198051533946i	
40.7389319658447 + 29.7594816102787i	
58.0000000000000 + 5.00000000000000i	
22.9179222836477 - 34.0983827659903i	
18.9913780286485 + 27.8198051533946i	
56.9222298353378 - 34.9245708620177i	
-29.0000000000000 + 0.00000000000000i	
56.9222298353378 + 34.9245708620177i	
18.9913780286485 - 27.8198051533946i	
22.9179222836477 + 34.0983827659903i	
58.0000000000000 - 5.00000000000000i	
40.7389319658447 - 29.7594816102787i	
-38.9913780286485 - 35.8198051533946i	
-4.57908408483015 + 127.066706485749i
```

在补零之后的序列计算结果为，对于16点的FFT结果*k* = 0*,* 2*,* 4*,* 6*,* 8*,* 10*,* 12*,* 14，对应于8点的FFT结果*k* = 0*,* 1*,* 2*,* 3*,* 4*,* 5*,* 6*,* 7。

例程：

```matlab
x1=[-1 -0.5 1 2 2 1 0.5 -1];
x2(16)=0;
x2(1:8)=x1;
subplot(221);
stem(0:1:7,x1);axis([-0.5 7.5 -1.5 2.5]);
ylabel('x(n) \rightarrow');title('8 samples');
subplot(222);
stem(0:1:7,abs(fft(x1)));axis([-0.5 7.5 -0.5 10]);
ylabel('|X(k)| \rightarrow');title('8-point FFT');
subplot(223);
stem(0:1:15,x2);axis([-0.5 15.5 -1.5 2.5]);
xlabel('n \rightarrow');ylabel('x(n) \rightarrow');
title('8 samples+zero-padding');
subplot(224);
stem(0:1:15,abs(fft(x2)));axis([-1 16 -0.5 10]);
xlabel('k \rightarrow');ylabel('|X(k)| \rightarrow');
title('16-point FFT');
```



## 短时傅里叶变换：

信号是连续的，在一直产生，但是信号处理只能处理离散傅里叶变化无法对连续的信号在PC上做处理，所以需要对信号进行截断，如截取N个数据做FFT操作。但是对数据进行截断操作在做FFT之后会产生频谱泄露。对数据加窗可以减少频谱泄露，加窗的主要作用是减弱因无穷级数截断而产生的吉布斯现象的影响。在音频中一般加入hanning窗或者hamming窗。

```matlab

N=8000;
FS=44100;
x=cos(2*pi*1000*(0:1:N-1)/44100)';
figure(2)
% W=blackman(N);
% W=N*W/sum(W); % scaling of window
% f=((0:N/2-1)/N)*FS;

W=hanning(N);
xw=x.*W;
subplot(2,2,1);plot(0:N-1,x);
axis([0 1000 -1.1 1.1]);
title('a) Cosine signal x(n)')
subplot(2,2,3);plot(0:N-1,xw);axis([0 8000 -4 4]);
title('c) Cosine signal x_w(n)=x(n) \cdot w(n) with window')
X=20*log10(abs(fft(x,N))/(N/2));
subplot(2,2,2);plot(f,X(1:N/2));
axis([0 10000 -80 10]);
ylabel('X(f)');
title('b) Spectrum of cosine signal')
Xw=20*log10(abs(fft(xw,N))/(N/2));
subplot(2,2,4);plot(f,Xw(1:N/2));
axis([0 10000 -80 10]);
ylabel('X(f)');
title('d) Spectrum with Hanning window')

```



常见的窗函数有：

| 窗          | matlab函数 | 数学公式                                               |
| ----------- | ---------- | ------------------------------------------------------ |
| Blackman 窗 | blackman   | $w(n)=0.42-0.5cos(2\pi n/(N-1))+0.08cos(4\pi n/(N-1))$ |
| Hamming窗   | hamming    | $w(n)=0.54-0.46cos(2\pi n/(N-1))$                      |
| Hanning窗   | hann       | $w(n)=0.5(1-cos(2\pi n/(N-1)))$                        |

窗函数的分析在matlab中使用`wvtool`这个函数进行查看，也可以通过windowDesigner进行窗函数设计。还有其他很多窗函数没有使用暂不做说明。

在语音信号处理中，数据的重叠最好是75%，这样在加窗之后不会产生泄露？(忘记原因了)，在实际使用中一般重叠为50%，数据的帧长和帧移的关系是帧移为帧长一半，实际使用中发现有hanning窗和sqrt_hanning窗，两种窗函数的加窗和去窗方式不一样，但是实际效果是一样的。

## STFT的matlab代码：

```matlab
clear;

wav_in='./wav/in.wav';
wav_out='./wav/hanning.wav';

[x,fs]=audioread(wav_in);

ms=16;
len=fs*ms/1000;
prec=50;
len1=floor(len*prec/100);%reserve data length
len2=len-len1;%refresh data length

win=hanning(len);
nFFT=len; %256

x_old=zeros(len1,1);
nframes=floor(length(x)/len2)-floor(len/len2);
xfinal=zeros(nframes*len2,1);

k=1;
for n=1:nframes
    in_buf=win.*x(k:k+len-1);
    spec=fft(in_buf,nFFT);
    
    
    out_buf=real(ifft(spec,nFFT));
    xfinal(k:k+len2-1)=out_buf(1:len1)+x_old;
    x_old=out_buf(len1+1:len);
    k=k+len2;
end

audiowrite(wav_out,xfinal,fs);


wav_out='./wav/sqrt_hanning.wav';
win=sqrt(hanning(len));
xfinal=zeros(nframes*len2,1);
out=zeros(len,1);
k=1;
for n=1:nframes
    in_buf=win.*x(k:k+len-1);
    spec=fft(in_buf,nFFT);
    
    
    out_buf=real(ifft(spec,nFFT));
    out=out+win.*out_buf;
    xfinal(k:k+len2-1)=out(1:len1);
    out(1:len1)=out(len1+1:2*len1);
    out(len1+1:2*len1)=zeros(len1,1);
    k=k+len2;
end

audiowrite(wav_out,xfinal,fs);
```

## STFT的C语言代码：

**stft.h**

```c
#ifndef STFT_H
#define STFT_H

#define FS 16000
#define MS 16
#define LEN (MS * FS / 1000)
#define PREC (50)
#define LEN1 (LEN * PREC / 100) // reserve data length
#define LEN2 (LEN - LEN1)       // refresh data length
#define NFFT (LEN)
#define WIN_LEN (LEN)

void STFT_Init(void);
void STFT_Process(short *in, short *out);
#endif // !STFT_H
```

**stft.c**

```c
#include "stft.h"
#include "fft2.h"
#include <math.h>
#include <stdio.h>
#include <string.h>

static float win[WIN_LEN] = {0.0f};
static float x_old[LEN1] = {0.0f};
static float combine[LEN] = {0.0f};
static Complex fft_in[LEN] = {0.0f};
static short empty = 1;

void STFT_Init(void)
{
    for (short i = 0; i < WIN_LEN; i++)
    {
        /*这里使用Hamming窗*/
        win[i] = 0.54f - 0.46f * cos(2 * M_PI * i / (WIN_LEN - 1));
    }
}

void STFT_Process(short *in, short *out)
{
    if (empty)
    {
        empty = 0;
        for (short i = 0; i < LEN2; i++)
        {
            combine[LEN1 + i] = in[i];
        }
        memset(out, 0, LEN2 * sizeof(short));
        return;
    }
    memcpy(&combine[0], &combine[LEN2], LEN1 * sizeof(float));
    for (short i = 0; i < LEN2; i++)
    {
        combine[LEN1 + i] = in[i];
    }
    for (short i = 0; i < LEN; i++)
    {
        fft_in[i].real = combine[i] * win[i];
        fft_in[i].imag = 0.0f;
    }
	/*采样复数fft*/
    fft(fft_in, NFFT);

    for (short i = 1; i < 10; i++)
    {
        fft_in[i].real = fft_in[i].real * 0.1;
        fft_in[i].imag = fft_in[i].imag * 0.1;

        fft_in[NFFT - i].real = fft_in[NFFT - i].real * 0.1;
        fft_in[NFFT - i].imag = fft_in[NFFT - i].imag * 0.1;
    }

    ifft(fft_in, NFFT);

    /*output*/
    for (short i = 0; i < LEN2; i++)
    {
        out[i] = x_old[i] + fft_in[i].real;
        x_old[i] = fft_in[i + LEN2].real;
    }
}
```

