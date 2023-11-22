---
title: 控件的API接口函数
author: lixinghui
date: 2023-3-27 12:00:00 +0800
categories: [Labwindows]
tags: [Labwindows]
---


labwindows的控件繁多，每个控件的API接口函数也不少，如何快速的找到想要使用的API接口函数呢？labwindows的help提供了很大的便利。

## 标准控件函数查看

以RING控件为例，右键先放置一个RING的控件，选中该控件，右键选择`Control Help`，会有五个选项，分别为:

>   `Operating Ring Controls`操作RING控件
>   `Programming Ring Controls`编程RING控件
>   `Ring Control Attributes`RING控件属性
>   `Ring Control Functions`RING控件函数
>   `Ring Control Events`RING控件事件

其中对于编程来说一般查看的为`Programming Ring Controls`和`Ring Control Functions`这两项，可以查看所有的API接口函数。

## 查看示例

如果一个函数比较复杂，单查看帮助依然看不懂如何使用，可以点进去该函数的详情页，他会详细讲解函数每个输入输出的意义，同时最下面很可能有示例，点进去示例就可以更加清晰明了的了解函数如何使用了。



## 打开/关闭串口

```c
int OpenComConfig (int Port_Number, char Device_Name[], long Baud_Rate, int Parity, int Data_Bits, int Stop_Bits, int Input_Queue_Size, int Output_Queue_Size);
int OpenCom (int Port_Number, char Device_Name[]);
int CloseCom (int Port_Number);

/*示例*/
ret_val = OpenComConfig(com_port, "", baud_rate, 0, 8, 1, 2048, 2048);
if (ret_val < 0)
{
    return -1;
}
/* 	Turn off Hardware handshaking (loopback test will not function with it on) */
SetCTSMode(com_port, LWRS_HWHANDSHAKE_OFF);
/* Sets timeout limit for input/output operations. */
SetComTime(com_port, 2.0);
```

打开串口一般使用`OpenComConfig`函数，该函数可以配置并打开串口，所以`OpenCom`使用较少。关于`OpenComConfig`函数的详细解析查看labwindows的Help文档或者查看[LabwindowsCVI_4](https://annie-timor.github.io/2023/03/27/LabwindowsCVI_4/)博客。

## 建立串口中断回调函数

```c
int InstallComCallback (int portNumber, int eventMask, int notifyCount, int eventCharacter, ComCallbackPtr callbackFunction, void *callbackData);

/*示例*/
InstallComCallback (Com_port, LWRS_RECEIVE, 7, 0, Event_Char_Func, 0);    //绑定串口事件回调函数 
```

对于建立回调函数，最重要的就是触发条件，如果设置每接收到字符都触发选择`LWRS_RXCHAR`，如果以固定字符作为结束符则选择`LWRS_RXFLAG`，接收固定长度的数据触发中断则选择`LWRS_RECEIVE`，多条件同时触发使用`|`来进行连接。详细解析查看labwindows的Help文档或者查看[LabwindowsCVI_4](https://annie-timor.github.io/2023/03/27/LabwindowsCVI_4/)博客。

## 清空串口缓冲区

```c
 FlushInQ(com_port);
 FlushOutQ(com_port);
```



## 开机自运行函数

在labwindows中并没有自启动回调函数，那么就把需要开机运行的函数放入主函数之间，如下：

```c
int main(int argc, char *argv[])
{
    if (InitCVIRTE(0, argv, 0) == 0)
        return -1; /* out of memory */
    if ((panelHandle = LoadPanel(0, "main.uir", PANEL)) < 0)
        return -1;
    DisplayPanel(panelHandle);
    /*开机运行项*/
    ScanComPort();
    /*结束开机运行*/
    RunUserInterface();
    DiscardPanel(panelHandle);
    return 0;
}
```

## 选择文件/文件夹

```c
/*选择文件*/
int FileSelectPopup (char defaultDirectory[], char defaultFileSpec[], char fileTypeList[], char title[], int buttonLabel, int restrictDirectory, int restrictExtension, int allowCancel, int allowMakeDirectory, char pathName[]);

int FileSelectPopupEx (char defaultDirectory[], char defaultFileSpec[], char fileTypeList[], char title[], int buttonLabel, int restrictDirectory, int restrictExtension, char pathName[]);

/*选择文件夹*/
int DirSelectPopup (char defaultDirectory[], char title[], int allowCancel, int allowMakeDirectory, char pathName[]);

int DirSelectPopupEx (char defaultDirectory[], char title[], char pathName[]);

/*示例*/
char path_name[512] = {0};
DirSelectPopupEx("", "请选择文件夹", path_name);  
FileSelectPopupEx("", "*.wav", "*.*", "请选择目标文件", VAL_SELECT_BUTTON, 0, 0, path_name);
```

这四个函数中，带Ex结尾的比不带的使用简单，和更好用一些（打开文件夹的不带Ex有点不符合windows的一般操作）
