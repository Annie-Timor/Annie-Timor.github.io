---
title: 声压级相关
author: lixinghui
date: 2024-1-2 9:00:00 +0800
categories: [Speech, SPL]
tags: [语音杂谈]
math: true
---

## 一、声压级相关

### 1、声压级

1、人耳可听的声压范围为$$2*10^{-5}pa$$~$$20pa$$之间，对应到声压级为`0dB~120dB`

2、线性尺度和分贝尺度：

```
    1pa  <--->  0dB
0.001pa  <--->  -60dB
20*log10(线性尺度)=分贝尺度
10^(分贝尺度/20)=线性尺度
```

3、人的大脑对于瞬时声压幅值波动没有响应，但对动态声压均方根(RMS)有响应，平均响应时间间隔为35ms

$$
p=\sqrt{\frac{1}{T}\int_{0}^{T}(p_{var}(t))^2dt}
$$

其中$$p_{var}$$为波动声压的瞬时幅值。

针对一个纯音信号，均方根值等于幅值的`0.707`倍

4、声压级计算公式：

$$
SPL(dB)=20*log10(\frac{p}{p_{ref}})=10*log10(\frac{p^2}{p^2_{ref}})
$$

其中$$p_{ref}$$为人耳可听最小声压幅值`20uPa`，这里的声压为均方根值(RMS)

5、声压级的大小及对应点

```
  0dB  <--->  20uPa  <--->  可听阈1KHz
 94dB  <--->  1Pa    <--->  校准值
100dB  <--->  2Pa    <--->  加倍
114dB  <--->  10Pa   <--->  校准
120dB  <--->  20Pa   <--->  痛阈
```

当然`120dB`上面肯定还有更大的声压，这里不进行讨论

### 2、声压级的计算

1、使用声压传感器测量得出时域信号，即声压幅值随时间的变化曲线；

2、用于计算的声压级一定是声压的均方根值，而非瞬时声压的幅值，并且在整个频带上的总有效值，也就是overall level；

例如：overall level为`1.97pa(A加权)  <--->  99.88dB(A)`

### 3、麦克风的灵敏度

例如以一个模拟麦克风的参数为例：

```
测试条件：94dB SPL @ 1KHz   -42dBV/Pa
```

根据模拟麦克风的计算公式：

$$
Sensitivity_{dBV}=20*log10(\frac{Sensitivity_{mv/Pa}}{Output_{AREF}})
$$

其中$$Output_{AREF}$$为`1000mV/Pa(1V/Pa)`，为参考输出比；

将灵敏度的结果带入进去，计算得到$$Sensitivity_{mv/Pa}=7.943mv/Pa$$;

也就是说在`94dB`的输入情况下，麦克风的输出为`-42dB`，对应到电压为`7.943mv(RMS)`

将该麦克风的输入放大`20dB`，输入为`114dB`，输出为`-22dB`，对应到电压值为 `79.43mv(RMS)`；

再放大`20dB`，输入为`134dB`，输出为`-2dB`，对应到电压为`794.3mv(RMS)`；

当然，该麦克风的AOP(最大输入声压级)为`130dB`，这里的`134dB`的输入超过了最大输入范围。

2、理解灵敏度和信噪比的关系

```
       Sensivity=-42dBV
       SNR=59dB
       AOP=130dB
             ┌─
    130─┐----│- -6
        │    │
    120─┤----├─ -16
        │   2│
I       │   6│       O
N       │    │       U
P    94─┤----├─ -42  T
U       │    │       P
T       │   S│       U
        │   N│       T
        │   R│
        │    │
     35─┤----└─ -101dB
        │
   0dB ─┘
   
   根据灵敏度，94dB输入时，输出为-42dB，换为正数为78dB=7.943mV
   根据信噪比，输入为35dB时，输出为-101dB，换为正数为19dB=8.912uV,低于35dB无反应
   根据最大输入声压级，输入为130dB时，输出为-6dB,换为正数为114dB=501.169mV
   上面这个图将输入和输出的关系对上了。
   这里仅仅是麦克风的输出电压，转换到数字信号还需要考虑到ADC的影响。
```

关于数字麦克风请参考ADI的文章，[《理解麦克风的灵敏度》](./理解麦克风的灵敏度.pdf)

### 4、数字信号的计算

这里假设麦克风的输出为`0dB ~ -120dB`，反过来也就是`0dB ~ 120dB`,设定标尺为`1e-6`这个值；

声压级的计算为`20*log10(rms/1e-6)`，当`rms=1`的时候，最大输出声压级为`120dB`，当然这里的`rms<1`，因为数字信号做归一化之后最大值为1，连续信号的均方根值肯定小于1的；

生成一段数字信号：

```
N=1024;
x=rand(1024,1)./100;
```

时域声压级的计算：

```
trms=sqrt(sum(x.^2)/N);
tspl=20*log10(trms/1e-6);
```

频域声压级的计算：

```
y=abs(fft(x,N))./N;
frms=sqrt(sum(y.^2));
fspl=20*log10(frms/1e-6);
```

频域每个频点的声压级计算：

```
y=abs(fft(x,N))./N;
prms=sqrt(y.^2);
pspl=20*log10(prms./1e-6);
```



### 5、声压级计算比较

```matlab
%% 信号初始化
FS=48000;

[data,~]=audioread('wave1.wav');

N=length(data);
M=2048;
cnt=floor(N/M);



%% 时域计算
tspl=zeros(cnt,1);
for i=1:1:cnt
    x=data((i-1)*M+1:i*M);
    trms=sqrt(sum(x.^2)/M);
    tspl(i)=20*log10(trms/1e-6);
end

fileID = fopen('time_domain_spl.txt', 'w');
fprintf(fileID, '%f\n', tspl);
fclose(fileID);

%% 频域计算
fspl=zeros(cnt,1);
for i=1:1:cnt
    x=data((i-1)*M+1:i*M);
    y=abs(fft(x,M))./M;
    frms=sqrt(sum(y.^2));
    fspl(i)=20*log10(frms/1e-6);
end

fileID = fopen('frequency_domain_spl.txt', 'w');
fprintf(fileID, '%f\n', fspl);
fclose(fileID);


%% 频域加窗计算
wspl=zeros(cnt,1);
for i=1:1:cnt
	% 这里没有计算完，每次只更新M/2个数据
    if i==1
        x=data(1:M);
    else
        x=data((i-1)*M/2+1:(i+1)*M/2);
    end
    x=x.*win;
    y=1.573*abs(fft(x,M))./M;
    frms=sqrt(sum(y.^2));
    wspl(i)=20*log10(frms/1e-6);
end
fileID = fopen('frequency_windows_spl.txt', 'w');
fprintf(fileID, '%f\n', wspl);
fclose(fileID);

%% A加权频域计算
aspl=zeros(cnt,1);
gain_scl=Ascale_calculate(FS,M);
for i=1:1:cnt
    x=data((i-1)*M+1:i*M);
    y=abs(fft(x,M))./M;
    y=y.*gain_scl;
    frms=sqrt(sum(y.^2));
    aspl(i)=20*log10(frms/1e-6);
end

fileID = fopen('frequency_a_weight_spl.txt', 'w');
fprintf(fileID, '%f\n', aspl);
fclose(fileID);

%% C加权频域计算
cspl=zeros(cnt,1);
gain_scl=Cscale_calculate(FS,M);
for i=1:1:cnt
    x=data((i-1)*M+1:i*M);
    y=abs(fft(x,M))./M;
    y=y.*gain_scl;
    frms=sqrt(sum(y.^2));
    cspl(i)=20*log10(frms/1e-6);
end

fileID = fopen('frequency_c_weight_spl.txt', 'w');
fprintf(fileID, '%f\n', cspl);
fclose(fileID);


%% A加权函数
function [A,dBA] = aWeight(f)
f1=20.60;
f2=107.7;
f3=737.9;
f4=12194;
A_1000=2;

A_=(f4^2*f^4)/((f^2+f1^2)*(f^2+f2^2)^0.5*(f^2+f3^2)^0.5*(f^2+f4^2));

dBA=20*log10(A_)+A_1000;
A=10^(dBA/20);
end

%% C加权函数
function [C,dBC] = cWeight(f)
    f1=20.60;
    f4=12194;
    C_1000=0.06;
    
    C_=(f4^2*f^2)/((f^2+f1^2)*(f^2+f4^2));
    
    dBC=20*log10(C_)+C_1000;
    C=10^(dBC/20);
end

%% A加权尺度变换
function scl=Ascale_calculate(fs,len)
    scl=zeros(len,1);
    gap=fs/len;
    
    scl(1)=1;
    for i=2:1:len/2
        freq=gap*(i-1);
        [scl(i),~]=aWeight(freq);
        scl(len+2-i)=scl(i);
    end
    scl(1+len/2)=aWeight(fs/2);
end

%% C加权尺度变换
function scl=Cscale_calculate(fs,len)
    scl=zeros(len,1);
    gap=fs/len;
    
    scl(1)=1;
    for i=2:1:len/2
        freq=gap*(i-1);
        [scl(i),~]=cWeight(freq);
        scl(len+2-i)=scl(i);
    end
    scl(1+len/2)=cWeight(fs/2);
end

```

原因：在没有加窗的时候，对无限长的数据进行截断，会导致频谱泄露和栅栏现象，旁瓣能量较高。

更改方式：

①、由原来的计算声压级的入口2048点改为1024点。

②、对输入数据做50%的混叠，两帧数据拼成2048点。

③、对拼帧完成的数据做加窗处理，加hamming窗就可以了。

④、加窗完成之后做FFT，做2048点的FFT。

⑤、在做了加窗之后，计算出来的声压级小于时域计算的，做一定的补偿，在加入的是2048点hamming窗时，相差3.937dB  ==>  尺度变换为1.573
