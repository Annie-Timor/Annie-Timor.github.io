---
title: 音频倍频程
author: lixinghui
date: 2023-3-27 12:00:00 +0800
categories: [Speech, octave]
tags: [语音杂谈]
math: true
---


倍频程的英文为Octave Band，其中Octave为八度音阶，关于八度音阶在Octave的维基百科中有详细说明，由此可见倍频程实际是从音乐的音阶中过来的。如何理解倍频程呢，1倍频，1/2倍频，1/3倍频程又是什么呢，根据中文的说文解字，“倍“就是翻倍的意思，也就是下一个频率为当前频率的一倍，上一个频率是当前频率的一半。

所以倍频的计算公式为：

$$
f_{n+1}=f_n*2
$$

$$
设定中心频率为f_c,上限频率为f_u,下限频率为f_l
$$

$$
f_u=f_c*2^{1/2},f_l=f_c/2^{1/2}
$$

除开倍频程还有1/2倍频程和1/3倍频程，根据上述公式可以推出，1/2和1/3倍频程的公式为：

$$
f_{n+1}=f_n*2^{1/2}
$$

$$
f_u=f_c*2^{1/2/2},f_l=f_c/2^{1/2/2}
$$

$$
f_{n+1}=f_n*2^{1/3}
$$

$$
f_u=f_c*2^{1/3/2},f_l=f_c/2^{1/3/2}
$$

以上计算公式都是以2为底的，1/2倍频程很少用到，在计算1/3倍频程的时候还有一种算法是以10为底的，说是基数为10更加合理一些，具体原理没有查阅，但是国标的数值确实更接近基10的数据一些，基10的计算公式如下：

$$
f_{n+1}=f_n*10^{1/10}
$$

$$
f_u=f_c*10^{1/10/2},f_l=f_c/10^{1/10/2}
$$

两者计算结果差距不大，$$2^{1/3}=1.2599210498948732,10^{1/10}=1.2589254117941673$$

两者的matlab代码如下

```matlab
%% Calculate Third Octave Bands (base 2) in Matlab
fcentre  = 10^3 * (2 .^ ([-18:13]/3)) % base frequence is 1000
fd = 2^(1/6);
fupper = fcentre * fd
flower = fcentre / fd

%% Calculate Third Octave Bands (base 10) in Matlab
fcentre = 10.^(0.1.*[12:43]) % [30-18,30+13]
fd = 10^0.05;
fupper = fcentre * fd
flower = fcentre / fd
```



国标对倍频程进行了标准化，具体结果如下：

![标准倍频程](https://pic4.zhimg.com/v2-6a863a5a20759adfcfa5b32175416e0f_b.jpg)

倍频程的中心频率，是由1000Hz这个频率计算得出的。

参考链接：

[一文读懂三分之一倍频程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/396189927)

[什么是倍频程? - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/26732433)

[Octave](https://en.wikipedia.org/wiki/Octave)

[Octave_band](https://en.wikipedia.org/wiki/Octave_band)







