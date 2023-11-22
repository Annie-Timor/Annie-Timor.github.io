---
title: C语言格式输出符
author: lixinghui
date: 2023-2-20 12:00:00 +0800
categories: [C语言, Output]
tags: [C语言]
---




## 1、字符、字符串

`%c`，输出字符，按照ASCII码进行输出。

`%s`，输出字符串，以`'\0'`作为结束符。

>   如果`%s`输出的字符串没有结束符则会输出后面不属于该字符串的字符
{: .prompt-warning }

```c
#include <stdio.h>
#include <string.h>


int main(int argc, char **argv)
{
    char *s1 = "hello world";
    char s2[] = "hello world";
    char s3[11] = "hello world";

    /*strlen function not incorporate the last character '\0' */
    printf("sizeof s1 = %d,length s1 = %d\n", sizeof(s1), strlen(s1));
    printf("sizeof s2 = %d,length s2 = %d\n", sizeof(s2), strlen(s2));
    printf("sizeof s3 = %d,length s3 = %d\n", sizeof(s3), strlen(s3));

    printf("%s\n", s1);
    printf("%s\n", s2);
    printf("%s\n", s3);

    return 0;
}

/* output
sizeof s1 = 8,length s1 = 11
sizeof s2 = 12,length s2 = 11
sizeof s3 = 11,length s3 = 22
hello world
hello world
hello worldhello world
*/
```

>   如果已知字符串的长度，可以使用`%ns`来输出，n为字符串的长度，这里为`%11s`
{: .prompt-tip }

## 2、回车、换行

`\r`，回车符，归位键，使得光标回到起始位置（在windows中不会换行，只会回到该行行首）

`\n`，换行符，切换到下一行

基于`\r`的原理，可以制作出简单的进度条，如下：

```c
#include <Windows.h>
#include <stdio.h>
#include <string.h>

void schedule(int cur_position, int total_size);

int main()
{
    int cnt = 50;
    int cur = 0;
    int total = 100;

    while (cnt >= 0)
    {
        Sleep(100);
        cur += 2;
        schedule(cur, total);
        cnt--;
    }
    printf("\ndone\n");
    return 0;
}

// 显示进度条 函数
void schedule(int cur_position, int total_size)
{
#define CHARACTER_SIZE 50

    float num = ((float)cur_position / (float)total_size) * CHARACTER_SIZE; // num决定进度条中“>”的个数（0~50）
    printf("\r");                                                           // 删除上一次输出的内容
    printf("running:[");
    for (int i = 0; i < 50; i++)
    {
        if (i < num) // 输出num个">"
            printf(">");
        else
            printf(" "); // 其他用空格填充
    }

    if (num > 49.5) // 防止(cur_position / total_size)不能被除尽
        num = 50;

    printf("]%% %.2f", num * (100 / CHARACTER_SIZE)); // 输出完成进度的百分比
}

/* output
PS F:\MyCspace\demo> .\a.exe
running:[>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]% 100.00
done
*/
```



## 3、整形输出

`%d`，十进制输出，使用最多，`%u`以十进制无符号数输出，`%ld`使用长整型输出，很多机器`long`型与`int`型都是32bit的数据，这两个输出也是一样的长度，更长的为`long long`型，输出格式为`%lld`。

`%x`，十六进制输出，直接输出会省缺，`%#x`在数据之前补全0x，`%02x`两字节输出，会保留前面的0值，`%d`输出也是这样的道理，`%08d`会保留前面的0。

`%o`，八进制输出，没用过。

```c
#include <stdio.h>

int main(int argc, char **argv)
{
    unsigned char a = 8;
    unsigned char b = 255;

    printf("a=%x, b=%x\n", a, b);
    printf("a=%#x, b=%#x\n", a, b);
    printf("a=0x%02x, b=0x%02x\n", a, b);

    return 0;
}
/* output
a=8, b=ff
a=0x8, b=0xff
a=0x08, b=0xff
*/
```

## 4、浮点数输出

`%f`，单精度浮点数输出，默认保留6位小数位，`%.2f`保留2位小数位，`%8.2f`总输出8位，2位小数位。

`%lf`，双精度浮点数输出，默认保留6位小数。

`%e`，以指数型式输出浮点数

`%g`，不输出无意义的0

```c
int main(int argc, char **argv)
{
    float a = 9876.31;
    float b = 3.45;

    printf("a = %f, b = %.2f\n", a, b);
    printf("a = %e, b = %8.2f\n", a, b);
    printf("a = %g, b = %g\n", a, b);

    return 0;
}
/* output
a = 9876.309570, b = 3.45
a = 9.876310e+003, b =     3.45
a = 9876.31, b = 3.45
*/
```

## 5、左右对齐

默认采用的是右对齐模式，在前面添加`-`可以转成左对齐，如`%-3d`为左对齐的3字节输出整型变量。

```c
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{

    for (short int i = 0; i < 64; i++)
    {
        printf("%-3d,", i);
        i % 8 == 7 ? printf("\n") : 0;
    }

    return 0;
}
/* output
0  ,1  ,2  ,3  ,4  ,5  ,6  ,7  ,
8  ,9  ,10 ,11 ,12 ,13 ,14 ,15 ,
16 ,17 ,18 ,19 ,20 ,21 ,22 ,23 ,
24 ,25 ,26 ,27 ,28 ,29 ,30 ,31 ,
32 ,33 ,34 ,35 ,36 ,37 ,38 ,39 ,
40 ,41 ,42 ,43 ,44 ,45 ,46 ,47 ,
48 ,49 ,50 ,51 ,52 ,53 ,54 ,55 ,
56 ,57 ,58 ,59 ,60 ,61 ,62 ,63 ,
*/
```

