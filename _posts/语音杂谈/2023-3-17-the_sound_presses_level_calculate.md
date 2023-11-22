---
title: 声压级计算
author: lixinghui
date: 2023-3-17 12:00:00 +0800
categories: [Speech, SPL]
tags: [语音杂谈]
math: true
---



## 声压级理论计算


声压级(Sound Pressure Level SPL)是用来描述声压的大小的，声压就是声音的压力波形，人耳可听的声压范围为$$ 20\mu Pa - 20Pa $$之间，也就是$$ 20*10^{-6}-20Pa $$，根据声压级计算公式$$ SPL(dB)=20*log_{10}(P/P_{ref}) $$，其中P表示平均声压，Pref表示参考压强，大小为$$ 20\mu Pa $$,可以得出，声压级的取值范围为0~120dB.

需要注意的是，传声器（麦克风）的校准一般使用的是94dB或者114dB，原因是因为该声压级对应的声压为1Pa和10Pa。使用麦克风的时候，一般为模拟麦或者数字麦，但是不管是模拟麦还是数字麦，给到计算的时候都是一帧帧的数据，如何理解这一帧的数字信号呢，下面详细讲解一下。
## 麦克风对应到声压级

>   1、模拟麦克风
>
>   翻看模拟麦克风的数据手册，与声压级计算有关的参数为灵敏度(Sensitivity)这一栏，以钰太的ZTS6556这颗麦克风为例，它的该项参数为-38dBV/Pa，测试条件为94dB SPL@1KHz。
>
>   根据模拟麦克风的计算公式：
>   
>   $$
>   Sensitivity(dBV)=20*log_{10}(\frac{Sensitivity(mV/Pa)}{Output(AREF)})
>   $$
>
>   <div align="center">
>       其中$Output(AREF)=1V/Pa=1000mV/Pa$ 
>   </div>    
>   <div align="center">将灵敏度参数带入计算：$-38=20*log_{10}(\frac{Sensitivity}{1000mV/Pa})$</div>
>   <div align="center">得：$Sensitivity=12.6mV/Pa$</div>
>
>   根据计算出来的结果可以知道在1Pa/94dB声压的时候，模拟麦克风输出的是12.6mV，在计算出这个值之后，我们可以设计麦克风输出到ADC检查之间的增益，在ADC之后的数据计算声压级的时候需要把$P/Pref$的这个P值量化到1Pa去。
>
>   ---
>
>   2、数字麦克风
>
>   数字麦克风同样也是看灵敏度(Sensitivity)这一栏，以敏芯微的MSM261SHT002这颗麦克风为例，它的该项参数为-26dBFS@1KHz 1Pa。假设选择16bit输出，则输出数据为-32768~+32767
>
>   根据数字麦克风的计算公式：
>   
>   $$
>   Sensitivity(dBFS)=20*log_{10}(\frac{Sensitivity(\%FS)}{Output(DREF)}) 
>   $$
>   
>   <div align="center">
>   其中$Output(DREF)为满量程数字输出$
>   </div>  
>   <div align="center"> 将灵敏度参数带入计算：$-26=20*log_{10}(\frac{Sensitivity}{32768})$</div>
>   <div align="center"> 得：$Sensitivity=1642$ </div>
>   所以94dB的声压的数据在1642左右。
>   
>   3、注意事项
>
>   需要注意的是，数字麦克风和模拟麦克风在峰值电平和均方根电平的使用上并不一致。麦克风的声学输入电平（单位为dB SPL）始终为均方根测量值，与麦克风的类型无关。模拟麦克风的输出以1 V rms为参考，因为均方根测量值更常用于比较模拟音频信号电平。然而，数字麦克风的灵敏度和输出电平却表示为峰值电平，因为它们是以满量程数字字（即峰值）为参考的。
>
>   [参考链接](https://www.analog.com/cn/analog-dialogue/articles/understanding-microphone-sensitivity.html)

## 采样出的数值计算声压级

声压级可分为时域和频域计算，但是一般是在时域内进行计算，频域一般是计算在不同频率下的能量分布，当然也可以计算是的。

时域计算：

```matlab

function spl=UserSPLTimeDomain(x,block_size,reference_pressure)

n=floor(length(x)/block_size);
spl=zeros(n,1);

rms=0;
sound_level=0;

for i=1:n
    x_=x((i-1)*block_size+1:i*block_size);
    rms=0;
    rms=sum(x_.^2);
    rms=sqrt(rms/block_size);
    
    sound_level=20*log10(rms/reference_pressure);
    spl(i)=sound_level;
end

end
```

```c
// 计算声压级函数
double calcSoundLevel(double *buffer, int length, double referencePressure)
{
    double rms = 0.0;
    double soundLevel = 0.0;

    // 计算均方根（RMS）
    for (int i = 0; i < length; i++)
    {
        rms += buffer[i] * buffer[i];
    }
    rms = sqrt(rms / length);
    
    // 计算声压级
    soundLevel = 20.0 * log10(rms / referencePressure);
    
    return soundLevel;
}
```



频域计算：

```matlab
function spl=UserSPLFreqDomain(x,block_size,reference_pressure)
n=floor(length(x)/block_size);
spl=zeros(n,1);
rms=0;
sound_level=0;
for i=1:n
    x_=x((i-1)*block_size+1:i*block_size);
    y_=abs(fft(x_,block_size))/block_size;
    
    
    energy=sum(y_.^2);
    %rms=sqrt(energy/block_size);
    rms=sqrt(energy);
    
    sound_level=20*log10(rms/reference_pressure);
    spl(i)=sound_level;
end
end
```

## A加权计算
A 加权应用于仪器测量的声级，以解释人耳感知的相对响度，因为耳朵对低音频频率不太敏感。它通过算术方式将按倍频程或第三倍频程列出的值表添加到以 dB 为单位的测量声压级中。通常添加所得的倍频程波段测量值（对数法）以提供描述声音的单个 A 加权值;单位写为 dB（A）或者是dBA。
A加权虽然最初仅用于测量低电平声音（约40 phon），但现在通常用于测量环境噪声和工业噪声，以及评估潜在的听力损伤和其他噪声健康影响 在所有声级;事实上，现在所有这些测量都必须使用A频率加权。

公式：

$$
R_A(f)=\frac{12194^2f^4}{(f^2+20.6^2)\sqrt{(f^2+107.7^2)(f^2+737.9^2)}(f^2+12194^2)} 
$$

$$
A(f)=20log_{10}(R_A(f))-20log_{10}(R_A(1000)) 
$$

$$
=20log_{10}(R_A(f))+2
$$

频域A加权计算：

```matlab

function spl=UserSPLFreqDomain_aWeight(x,block_size,fs,reference_pressure)
n=floor(length(x)/block_size);
spl=zeros(n,1);
rms=0;
sound_level=0;
for i=1:n
    x_=x((i-1)*block_size+1:i*block_size);
    y_=abs(fft(x_,block_size))/block_size;
    gain=zeros(block_size,1);
    for j=0:block_size/2
        freq=j*fs/block_size;
        [gain(j+1),~]=aWeight(freq);
        if j~=0 && j~=block_size/2
            gain(block_size-j+1)=gain(j+1);
        end
    end
    y_=y_.*gain;
    energy=sum(y_.^2);
    %rms=sqrt(energy/block_size);
    rms=sqrt(energy);
    
    sound_level=20*log10(rms/reference_pressure);
    spl(i)=sound_level;
end
end

function [A,dBA] = aWeight(f)
f1=20.60;
f2=107.7;
f3=737.9;
f4=12194;
A_1000=-2;

A_=(f4^2*f^4)/((f^2+f1^2)*(f^2+f2^2)^0.5*(f^2+f3^2)^0.5*(f^2+f4^2));

dBA=20*log10(A_)-A_1000;
A=10^(dBA/20);

end

```

[A加权算法链接](https://en.wikipedia.org/wiki/A-weighting)

[一文读懂A计权 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/400171691)
