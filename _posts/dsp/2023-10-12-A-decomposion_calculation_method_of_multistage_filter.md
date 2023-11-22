---
title: 一种多阶滤波器分解计算的方式
author: lixinghui
date: 2023-10-12 14:10:00 +0800
categories: [DSP, filter]
tags: [数字信号处理]
math: true
render_with_liquid: false
---

​	在单片机的应用中，对于很多的单片机在计算双精度浮点数会比单精度浮点数慢很多，一些单片机也有单精度的浮点数计算单元，对于这些个单片机在计算双精度数据的时候就会比较慢，如果能不使用双精度的浮点数就能计算完成的话，就会很好。

​	对于一些滤波器来说，特别是高阶的IIR滤波器，通过多级的级联，会导致滤波器的参数比较敏感，对于一点的变动就可能导致滤波器不稳定，从而数据无法收敛，对于高阶滤波器尤为如此！！！。

这里以一个例子来说，设计一个滤波器要求采样率为48000，带通为108~145Hz。

这里使用matlab的filterdesigner来设计滤波器。

选择巴特沃斯的IIR滤波器，将参数输入进去，阶数选择为4阶，点击Designer Filter。

这时等待滤波器设计完成，设计完成之后使用`ctrl+e`导出数据，导出系数为`SOS,G`；

使用函数`sos2tf`将参数转换为分子分母的形式，`[bb,aa]=sos2tf(SOS,G);`

得到：

```
bb=[5.84433468724046e-06, 0, -1.16886693744809e-05, 0, 5.84433468724046e-06];
aa=[1, -3.99261485225624,5.97840556730783, -3.97896460275009, 0.993173959450330];
```

这里的参数`bb`乘上的系数都为`e-6`的级别了，而单精度浮点数最大的精度大致为6~9位之间，双精度浮点数在15~17位之间。

将滤波器使用到C语言代码中：

```c
const double bbd[] = {5.84433468724046e-06, 0, -1.16886693744809e-05, 0, 5.84433468724046e-06};
const double aad[] = {1, -3.99261485225624, 5.97840556730783, -3.97896460275009, 0.993173959450330};

short UserFilterDouble(short *x, short *y, short len)
{
    short i, j;
    double sum;
    static double hisx[5] = {0};
    static double hisy[5] = {0};

    for (i = 0; i < len; i++)
    {
        hisx[0] = x[i];
        sum = bbd[0] * hisx[0] + bbd[1] * hisx[1] + bbd[2] * hisx[2] +
              bbd[3] * hisx[3] + bbd[4] * hisx[4] - aad[1] * hisy[1] -
              aad[2] * hisy[2] - aad[3] * hisy[3] - aad[4] * hisy[4];
        hisy[0] = sum;

        // memcpy(hisx + 1, hisx, 4 * sizeof(double));
        // memcpy(hisy + 1, hisy, 4 * sizeof(double));
        hisx[4] = hisx[3];
        hisx[3] = hisx[2];
        hisx[2] = hisx[1];
        hisx[1] = hisx[0];

        hisy[4] = hisy[3];
        hisy[3] = hisy[2];
        hisy[2] = hisy[1];
        hisy[1] = hisy[0];
        y[i] = hisy[0] + 0.5;
    }
    return 0;
}

const float bbf[] = {5.84433468724046e-06, 0, -1.16886693744809e-05, 0, 5.84433468724046e-06};
const float aaf[] = {1, -3.99261485225624, 5.97840556730783, -3.97896460275009, 0.993173959450330};

short UserFilterFloat(short *x, short *y, short len)
{
    short i, j;
    float sum;
    static float hisx[5] = {0};
    static float hisy[5] = {0};

    for (i = 0; i < len; i++)
    {
        hisx[0] = x[i];
        sum = bbf[0] * hisx[0] + bbf[1] * hisx[1] + bbf[2] * hisx[2] +
              bbf[3] * hisx[3] + bbf[4] * hisx[4] - aaf[1] * hisy[1] -
              aaf[2] * hisy[2] - aaf[3] * hisy[3] - aaf[4] * hisy[4];
        hisy[0] = sum;

        // memcpy(hisx + 1, hisx, 4 * sizeof(float));
        // memcpy(hisy + 1, hisy, 4 * sizeof(float));
        hisx[4] = hisx[3];
        hisx[3] = hisx[2];
        hisx[2] = hisx[1];
        hisx[1] = hisx[0];

        hisy[4] = hisy[3];
        hisy[3] = hisy[2];
        hisy[2] = hisy[1];
        hisy[1] = hisy[0];
        y[i] = hisy[0] + 0.5;
    }
    return 0;
}
```

通过测试可以得出，浮点数的滤波器无法正常工作，无法稳定。

在filterdesigner生成的滤波器中，SOS为N*6的大小，每一行的SOS和G(N)为一组二阶滤波器。

可以使用:

```
[b1,a1]=sos2tf(SOS(1,:),G(1));
[b2,a2]=sos2tf(SOS(2,:),G(2));

%{ SOS
1	1.93791172761411	1	1	-1.03142542806645	0.399770404178568
1	1.66950057344938	1	1	-0.556757555007605	0.770358376736369
}%

%{ G
1.63920005383353
0.0168199893463193
1
}%
[b1,a1]=sos2tf(SOS(1,:),G(1));
b1=SOS(1,1:3)*G(1)={1.63920005383353	3.17662500822969	1.63920005383353}
a1=SOS(1,4:6)={1	-1.03142542806645	0.399770404178568}
```

```matlab
hz=300;
start_hz=100;
stop_hz=500;

[M,N]=size(SOS);

fileID = fopen('output.txt', 'w');
fprintf(fileID,"/* butterworth biquad filter; center freq:%dhz, " + ...
    "[%dhz ~ %dhz] */\n",hz,start_hz,stop_hz);
for i=1:1:M
    [b,a]=sos2tf(SOS(i,:),G(i));
    fprintf(fileID,"const float hz%d_b%d[]={%15.12f, %15.12f, %15.12f};\n",hz,i,b(1),b(2),b(3));
    fprintf(fileID,"const float hz%d_a%d[]={%15.12f, %15.12f, %15.12f};\n",hz,i,a(1),a(2),a(3));
end
fclose(fileID);
```

将每一个二阶滤波器转换出来。

在滤波的时候，数据依次进入每一个二阶滤波器进行处理，如下：

```c
const float b1[] = {0.002417505882, 0.000000000000, -0.002417505882};
const float a1[] = {1.000000000000, -1.995894131978, 0.996224472708};
const float b2[] = {0.002417505882, 0.000000000000, -0.002417505882};
const float a2[] = {1.000000000000, -1.996720720278, 0.996937925798};

short UserFilterBiquad(short *x, short *y, short len)
{
    short i, j;
    float sum;
    static float hisx1[3] = {0};
    static float hisy1[3] = {0};
    static float hisx2[3] = {0};
    static float hisy2[3] = {0};

    float *x_ = malloc(sizeof(float) * len);

    for (i = 0; i < len; i++)
    {
        hisx1[0] = x[i];
        sum = b1[0] * hisx1[0] + b1[1] * hisx1[1] + b1[2] * hisx1[2] +
              -a1[1] * hisy1[1] - a1[2] * hisy1[2];
        hisy1[0] = sum;

        hisx1[2] = hisx1[1];
        hisx1[1] = hisx1[0];

        hisy1[2] = hisy1[1];
        hisy1[1] = hisy1[0];
        x_[i] = hisy1[0];
    }

    for (i = 0; i < len; i++)
    {
        hisx2[0] = x_[i];
        sum = b2[0] * hisx2[0] + b2[1] * hisx2[1] + b2[2] * hisx2[2] +
              -a2[1] * hisy2[1] - a2[2] * hisy2[2];
        hisy2[0] = sum;

        hisx2[2] = hisx2[1];
        hisx2[1] = hisx2[0];

        hisy2[2] = hisy2[1];
        hisy2[1] = hisy2[0];
        y[i] = hisy2[0] + 0.5f;
    }
    free(x_);
    return 0;
}
```

经过这样处理之后，滤波器就可以使用单精度浮点数正常运行了。

当然，二阶滤波器的级联也可以组成4阶滤波器。

```
bb=conv(b1,b2);
aa=conv(a1,a2);
% 也可以使用series函数级联两个滤波器
[bb,aa]=series(b1,a1,b2,a2);
```

可以使用`fvtool`函数来查看生成滤波器的幅频响应。

```
fvtool(bb,aa);
```







