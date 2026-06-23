---
title: 窗函数补偿之后的修正技巧
author: lixinghui
date: 2024-11-15 14:10:00 +0800
categories: [DSP, window]
tags: [数字信号处理]
math: true
---


下面把一些主要窗函数的幅值恢复系数列，在下表中，表内 MATLAB 函数中的 N 是窗函数的长度。

| 窗函数名称 | MATLAB函数  | 幅值相等恢复系数 | 功率相等恢复系数 |
| ---------- | ----------- | ---------------- | ---------------- |
| 矩形窗     | boxcar(N)   | 1                | 1                |
| 汉宁窗     | hanning(N)  | 2                | 1.633            |
| 海明窗     | hamming(N)  | 1.852            | 1.586            |
| 布莱克曼窗 | blackman(N) | 2.381            | 1.812            |
|            |             |                  |                  |

```
clear all; clc; close all;
 
fs=1000;                         % 采样频率
N=1000;                          % 信号长度
t=(0:N-1)/fs;                    % 设置时间序列
f1=50; f2=65.75;                 % 两信号频率
x=cos(2*pi*f1*t)+cos(2*pi*f2*t); % 设置信号
wind=hanning(N)';
X=fft(x.*wind);                  % 乘窗函数并FFT
Y=abs(X)*2/1000;                 % 计算幅值
freq=fs*(0:N/2)/1000;            % 设置频率刻度
[A1, k1]=max(Y(45:65));          % 寻求第1个信号的幅值
k1=k1+44;                        % 修正索引号
[A2, k2]=max(Y(60:70));          % 寻求第1个信号的幅值
k2=k2+59;                        % 修正索引号
Theta1=angle(X(k1));             % 计算信号f1的初始相角
Theta2=angle(X(k2));             % 计算信号f2的初始相角
Y1=Y*2;                          % 对加窗后的幅值进行修正
% 显示频率和幅值
fprintf('f1=%5.2f   A1=%5.4f   A11=%5.4f   Theta1=%5.4f\n',freq(k1),A1,A1*2,Theta1); 
fprintf('f2=%5.2f   A2=%5.4f   A21=%5.4f   Theta2=%5.4f\n',freq(k2),A2,A2*2,Theta2);

% 作图
subplot 211; plot(freq,Y(1:N/2+1),'k'); xlim([40 75]); 
line([0 100],[.5 .5],'color','k');
xlabel('频率/Hz'); ylabel('幅值'); title('(a)频谱图-幅值修正前');
subplot 212; plot(freq,Y1(1:N/2+1),'k'); xlim([40 75]); 
line([0 100],[1 1],'color','k');
xlabel('频率/Hz'); ylabel('幅值'); title('(b)频谱图-幅值修正后');
set(gcf,'color','w');
```



### 补偿因子的计算

在信号处理特别是滤波设计中，窗口函数（Window Function）用于减少或控制信号的频谱泄露（Spectral Leakage），在应用窗口函数后，信号的幅度会因为窗口函数的引入而减小。这是由于窗口函数对信号施加了一个权重，使得信号的能量分布发生了改变。为了补偿这一效应，可以引入补偿因子（Compensation Factor），使得窗口后的信号幅度保持与窗口前一致。

####  补偿因子的计算

补偿因子的计算基于窗口函数的平均值，即对整个窗口函数的系数求和后再除以窗口长度。一般来说，补偿因子为窗口函数系数的平均值的倒数。

#### 常见窗口函数的补偿因子

以下是几种常见窗口函数的补偿因子计算：

#### 1. Hanning 窗（Hann Window）

Hanning窗的定义为：
$$
w(n)=0.5(1−cos⁡(\frac{2πn}{N−1}))
$$

Hanning窗的补偿因子为：
$$
C_{Hann}=\frac{1}{Average(w(n))}=\frac{2}{\sum_{n=0}^{N−1}}w(n)=\frac{2}{N}
$$

因此，Hanning窗的补偿因子为 `2`。

#### 2. Hamming 窗（Hamming Window）

Hamming窗的定义为：
$$
w(n)=0.54−0.46cos⁡(\frac{2πn}{N−1})
$$

Hamming窗的补偿因子为：
$$
C_{Hamming}=\frac{1}{Average(w(n))}=\frac{1}{0.54}≈1.85185
$$


#### 补偿因子的应用

当你对信号应用窗口函数后，可以将补偿因子乘以窗口化后的信号，以恢复信号的原始幅度。这种方法在频域分析和滤波设计中非常重要，因为它确保了信号的能量在经过窗口化处理后不会发生明显的变化。

#### 实际应用中的补偿

在实践中，使用这些补偿因子时应注意不同窗口函数在不同应用场景下的影响。例如，在滤波设计中，某些情况可能并不需要严格的幅度补偿，而是关注信号的频谱特性。因此，补偿因子的使用应根据具体应用需求来决定。

功率相等补偿因子（Power Compensation Factor）用于确保在应用窗口函数后，信号的功率保持不变。因为功率与信号的幅度平方成正比，所以功率相等的补偿因子不同于幅值相等补偿因子。

### 功率相等补偿因子的计算

对于功率相等的补偿因子，计算公式如下：

$$
C_{\text{Power}} = \frac{1}{\sqrt{\frac{1}{N} \sum_{n=0}^{N-1} w^2(n)}}
$$

其中：
- \( w(n) \) 是窗函数的值。
- \( N \) 是窗的长度。

#### 常见窗口函数的功率补偿因子

#### 1. Hanning 窗（Hann Window）

Hanning窗的功率补偿因子计算如下：

$$
C_{\text{Hann, Power}} = \frac{1}{\sqrt{\frac{1}{N} \sum_{n=0}^{N-1} \left(0.5 \left(1 - \cos\left(\frac{2\pi n}{N-1}\right)\right)\right)^2}} = \sqrt{\frac{8}{3}} \approx 1.63299 
$$

#### 2. Hamming 窗（Hamming Window）

Hamming窗的功率补偿因子计算如下：

$$
C_{\text{Hamming, Power}} = \frac{1}{\sqrt{\frac{1}{N} \sum_{n=0}^{N-1} \left(0.54 - 0.46 \cos\left(\frac{2\pi n}{N-1}\right)\right)^2}} \approx 1.5874 
$$

#### 3. Blackman 窗（Blackman Window）

Blackman窗的功率补偿因子计算如下：

$$
C_{\text{Blackman, Power}} = \frac{1}{\sqrt{\frac{1}{N} \sum_{n=0}^{N-1} \left(0.42 - 0.5 \cos\left(\frac{2\pi n}{N-1}\right) + 0.08 \cos\left(\frac{4\pi n}{N-1}\right)\right)^2}} \approx 2.232
$$

#### 实际应用中的功率补偿

当你使用窗口函数时，如果需要确保信号的功率保持不变，则应使用功率相等的补偿因子。与幅值相等的补偿因子不同，功率相等的补偿因子用于确保信号的平均能量不会因为应用窗口函数而发生显著变化。

在频谱分析和滤波设计中，这一点尤为重要，因为信号的功率通常是与物理量（如能量或强度）直接相关的。确保功率不变能够保持分析结果的物理意义。



参考文献：MATLAB 数字信号处理 85 个实用案例精讲——入门到进阶；宋知用（编著）
