---
title: 简易无刷电机上位机控制
author: lixinghui
date: 2023-3-27 12:00:00 +0800
categories: [Labwindows]
tags: [Labwindows]
---


对于这个简易无刷电机控制器，选用labwindows来进行设计，通讯选择使用串口进行通讯，其主要功能为显示电机的实际转速和设定电机的转速，电机的运行状态，同时设定PID值。
该应用为刚接触labwindows时所设计的，整体的设计不够精致，但能够满足需求。
其界面如下：

借助这个示例主要想讲一下labwindows的串口及它的中断回调函数，在labwindows中，串口的设置主要通过`OpenComConfig`这个函数，回调函数通过`InstallComCallback`来建立。

## OpenComConfig函数

>目的:
>
>打开COM口，设置COM口参数。如果`inputQueueSize`或`outputQueueSize`介于1和29之间，`OpenComConfig`强制将其设置为30。
>
>`OpenComConfig`禁用`XON/XOFF`模式和`CTS`硬件握手。如果需要修改这些默认值，请参考`SetXMode`、`SetCTSMode`和`SetComTime`函数的描述。
>
>如果指定的端口已经打开，`OpenComConfig`将关闭该端口，然后重新打开。有关更多信息，请参阅`CloseCom`函数描述。
>
>使用控制面板设置端口硬件I/O地址和中断级别。
>
>您可以使用非标准波特率。所有波特率值都由comm驱动逐字解释。

函数原型：

```c
int OpenComConfig (int Port_Number, char Device_Name[], long Baud_Rate, int Parity, int Data_Bits, int Stop_Bits, int Input_Queue_Size, int Output_Queue_Size);
```

参数解析：

`portNumber`：表示要在其上操作的COM端口的数字，有效范围: 1—1,000

`deviceName`：要打开的COM端口的名称，作为ASCII字符串传递，在windows和linux中的名称不一样，windows一般为"COM1"，在linux中是"/dev/ttyS0"

`baudRate`：所选端口的波特率，默认值:9600波特，取值范围:110、150、300、600、1200、2400、4,800、9,600、14,400、19,200、28,800、38,400、56,000、57,600、115200、128,000、256,000。

`parity`：所选端口的校验模式，默认值:0-no奇偶校验。有效值0 =无奇偶校验、1 =奇奇偶校验、2 =偶数奇偶、3 =标记奇偶性、4 =空间校验。

`dataBits`：所选端口的数据位数。默认值:7位数据位，取值范围:5、6、7、8位数据位。

`stopBits`：所选端口的停止位数。默认值:1位停止位，取值范围:1 ~ 2位停止位。

`inputQueueSize`：所选端口的输入队列的大小。

`outputQueueSize`：所选端口的输出队列的大小。如果设置0，则为512，设置小于30，则为30，理论上没有上限，但是NI不建议超过32767。

返回值：错误则返回负值，表示错误码。

## InstallComCallback函数

>   目的：
>   允许您为特定COM端口安装同步回调函数。
>   在调用`InstallComCallback`之后调用`CloseCom`会自动卸载回调函数。如果您随后调用`OpenCom`并希望使用回调函数，则必须再次调用`InstallComCallback`。

函数原型：

```c
int InstallComCallback (int portNumber, int eventMask, int notifyCount, int eventCharacter, ComCallbackPtr callbackFunction, void *callbackData);
```

示例代码：

```c
notifyCount = 50; /* Wait for at least 50 bytes in queue. */
eventChar = 10; /* Wait for LF. */
eventMask = LWRS_RXFLAG | LWRS_TXEMPTY | LWRS_RECEIVE;
InstallComCallback (portNumber, eventMask, notifyCount, eventChar, ComCallback, NULL);


/* Callback Function */
void ComCallback(int portNumber, int eventMask, void *callbackdata)
{
    if (eventMask & LWRS_RXFLAG)
        printf("Received specified character\n");

    if (eventMask & LWRS_TXEMPTY)
        printf("Transmit queue now empty\n");

    if (eventMask & LWRS_RECEIVE)
        printf("50 or more bytes in input queue\n");
}
```

参数解析：

`portNumber`：表示要在其上操作的COM端口的数字。

`eventMask`：调用回调函数的事件。如果要禁用回调，则传递0。

| Bit  | Hex Value | COM Port Event                        | Constant Name | Description                                                  |
| ---- | --------- | ------------------------------------- | ------------- | ------------------------------------------------------------ |
| 0    | 0x0001    | Any character received.               | LWRS_RXCHAR   | Set when a character is received and placed in the input queue. |
| 1    | 0x0002    | Certain character in input queue.     | LWRS_RXFLAG   | Set when one or more instances of the event character are received and placed in  the input queue. The event character is specified in  **eventCharacter**. |
| 2    | 0x0004    | Transmit queue empty.                 | LWRS_TXEMPTY  | Set when the last character in the output queue is sent.     |
| 3    | 0x0008    | CTS changed state.                    | LWRS_CTS      | Set when the CTS (clear-to-send) line changes state.         |
| 4    | 0x0010    | DSR changed state.                    | LWRS_DSR      | Set when the DSR (data-set-ready) line changes state.        |
| 5    | 0x0020    | RLSD changed state.                   | LWRS_RLSD     | Set when the RLSD (receive-line-signal-detect) line changes state. |
| 6    | 0x0040    | BREAK received.                       | LWRS_BREAK    | Set when a break is detected on input.                       |
| 7    | 0x0080    | Line status error occurred.           | LWRS_ERR      | Set when a line-status error occurs. Line-status errors are CE_FRAME, CE_OVERRUN, and  CE_RXPARITY. |
| 8    | 0x0100    | Ring signal detected.                 | LWRS_RING     | Set to indicate that a ring indicator was detected.          |
| 15   | 0x8000    | **notifyCount** bytes in input queue. | LWRS_RECEIVE  | Set to detect when at least **notifyCount** bytes are in the input  queue. Once this event has occurred, it does not trigger again until the input  queue falls below and then rises back above **notifyCount** bytes. |

`notifyCount`：在将LWRS_RECEIVE事件发送给回调函数之前，输入队列必须包含的最小字节数。默认值:0，有效范围:0到输入队列大小。

`eventCharacter`：触发LWRS_RXFLAG事件的字符或字节值。默认值:0，有效范围:0 ~ 255。

`callbackFunction`：处理COM端口事件回调的用户函数名。回调函数格式为:`void CVICALLBACK CallbackFunctionName (int portNumber, int  eventMask, void *callbackData);`

`callbackData`：传递给回调函数的数据。

返回值：错误则返回负值，表示错误码。



对于建立回调函数，最重要的就是触发条件，如果设置每接收到字符都触发选择`LWRS_RXCHAR`，如果以固定字符作为结束符则选择`LWRS_RXFLAG`，接收固定长度的数据触发中断则选择`LWRS_RECEIVE`，其他的好像没怎么选择过。





全部代码如下

main.c

```c
#include <cvirte.h>
#include <userint.h>
#include "main.h"
#include <rs232.h>
#include <ansi_c.h>
#include <formatio.h>


#define Input_Queue_Size 512		 //最大接受数据
#define Output_Queue_Size 512		 //最大输出数据


#define TRUE	1
#define FALSE   0




static unsigned char SendBuf[7]={0};			 //发送的数据
static unsigned char ReceiveBuf[512]={0};		 //接收的数据

static int Com_port=4;				 //端口号
static long Baut_rate=115200;		 //波特率
static int Check_bit=0;				 //校验位
static int Data_bit=8;				 //数据位
static int Stop_bit=1;				 //停止位

static int Motor_rate=0;		     //电机的速度


static int panelHandle;				 //句柄
static char Uart_flag=0;             //串口开关标志位，1表示打开，0表示关闭


void StartUart(void);				 //开启串口
char ShowSpeed(unsigned char Buf3,unsigned char Buf4);	 //显示速度
char DataDeal(void);				 //数据处理
void CVICALLBACK   Event_Char_Func(int portNo,int eventMask,void * callbackData); 

int main (int argc, char *argv[])
{
	if (InitCVIRTE (0, argv, 0) == 0)
		return -1;	/* out of memory */
	if ((panelHandle = LoadPanel (0, "main.uir", PANEL)) < 0)
		return -1;
	
	
	DisplayPanel (panelHandle);

	RunUserInterface ();
	DiscardPanel (panelHandle);
	return 0;
}


void StartUart(void)
{
	/* 	GetCtrlVal */
	GetCtrlVal (panelHandle, PANEL_UART_NUM, &Com_port);
	GetCtrlVal (panelHandle, PANEL_BAUD_RATE, &Baut_rate);
	GetCtrlVal (panelHandle, PANEL_CHECK, &Check_bit);
	GetCtrlVal (panelHandle, PANEL_DATA_BIT, &Data_bit);
	GetCtrlVal (panelHandle, PANEL_STOP_BIT, &Stop_bit);
	/* 	Open and Configure Com port */
	OpenComConfig (Com_port, "", Baut_rate, Check_bit, Data_bit, Stop_bit, Input_Queue_Size, Output_Queue_Size);

	/* 	Turn off Hardware handshaking (loopback test will not function with it on) */
	SetCTSMode (Com_port, LWRS_HWHANDSHAKE_OFF);

	InstallComCallback (Com_port, LWRS_RECEIVE, 7, 0, Event_Char_Func, 0);    //绑定串口事件回调函数
	
	/* 	Make sure Serial buffers are empty */
	FlushInQ (Com_port);
	FlushOutQ (Com_port);
	Uart_flag=1;
	SetCtrlVal (panelHandle, PANEL_LED_UART, 1); 
}

void CVICALLBACK   Event_Char_Func(int portNo,int eventMask,void * callbackData)
{
	int	strLen;
	static char return_value=0;
	strLen = GetInQLen (Com_port);
	ComRd (Com_port, (char*)ReceiveBuf, strLen);
//	ComRd (Com_port, (char*)ReceiveBuf, 7); 
	
	return_value=DataDeal();
	if(return_value==TRUE)
		SetCtrlVal (panelHandle, PANEL_LED_2, 1);
	else
		SetCtrlVal (panelHandle, PANEL_LED_2, 0);
	
	FlushInQ (Com_port);
}


int CVICALLBACK quit (int panel, int control, int event,
					  void *callbackData, int eventData1, int eventData2)
{
	switch (event)
	{
		case EVENT_COMMIT:
			QuitUserInterface (0);
			break;
	}
	return 0;
}

int CVICALLBACK START_UART (int panel, int control, int event,
							void *callbackData, int eventData1, int eventData2)
{
	switch (event)
	{
		case EVENT_COMMIT:
			StartUart();
			break;
	}
	return 0;
}

int CVICALLBACK STOP_UART (int panel, int control, int event,
						   void *callbackData, int eventData1, int eventData2)
{
	switch (event)
	{
		case EVENT_COMMIT:
			CloseCom (Com_port);
			SetCtrlVal (panelHandle, PANEL_LED_UART, 0); 
			Uart_flag=0;
			break;
	}
	return 0;
}



int CVICALLBACK START_MOTOR (int panel, int control, int event,
							 void *callbackData, int eventData1, int eventData2)
{
	switch (event)
	{
		case EVENT_COMMIT:
			/* 报头 */
			SendBuf[0]=0xaa;
			
			/* 主从机 */
			SendBuf[1]=0x01;
			
			/* 命令行 */
			SendBuf[2]=0x01;  
			
			/* 内容 */
			SendBuf[3]=0x01;
			
			SendBuf[4]=0x00;
			
			/* 报尾 */
			SendBuf[5]=0x11;
			
			/* 校验位 */
			SendBuf[6]=SendBuf[0]+SendBuf[1]+SendBuf[2]+SendBuf[3]+SendBuf[4]+SendBuf[5];
			
			FlushInQ (Com_port);
			ComWrt (Com_port, (char*)SendBuf, 7);
//			SetCtrlVal (panelHandle, PANEL_LED, 1); 
			break;
	}
	return 0;
}

int CVICALLBACK STOP_MOTOR (int panel, int control, int event,
							void *callbackData, int eventData1, int eventData2)
{
	switch (event)
	{
		case EVENT_COMMIT:
			/* 报头 */
			SendBuf[0]=0xaa;
			
			/* 主从机 */
			SendBuf[1]=0x01;
			
			/* 命令行 */
			SendBuf[2]=0x01;  //
			
			/* 内容 */
			SendBuf[3]=0x00;  //关机

			SendBuf[4]=0x00;  //
			
			/* 报尾 */
			SendBuf[5]=0x11;
			
			/* 校验位 */
			SendBuf[6]=SendBuf[0]+SendBuf[1]+SendBuf[2]+SendBuf[3]+SendBuf[4]+SendBuf[5];
			
			FlushInQ (Com_port);
			ComWrt (Com_port, (char*)SendBuf, 7);
//			SetCtrlVal (panelHandle, PANEL_LED, 0);
			break;
	}
	return 0;
}


int CVICALLBACK SET_SPEED (int panel, int control, int event,
						   void *callbackData, int eventData1, int eventData2)
{
	switch (event)
	{
		case EVENT_COMMIT:
			GetCtrlVal (panelHandle, PANEL_MOTOR_SET_RATE, &Motor_rate);
			
			if((Motor_rate>0)&&(Motor_rate<500))
			{
				Motor_rate=500;
			}
			if((Motor_rate<0)&&(Motor_rate>-500))
			{
				Motor_rate=-500;
			}
			
			/* 报头 */
			SendBuf[0]=0xaa;
			
			/* 主从机 */
			SendBuf[1]=0x01;
			
			/* 命令行 */
			SendBuf[2]=0x02;  //
			
			
			/* 速度 */
			if(Motor_rate>=0)
			{
				SendBuf[3]=Motor_rate/256;
				SendBuf[4]=Motor_rate%256;
			}
			else
			{
				Motor_rate=0-Motor_rate;
				SendBuf[3]=Motor_rate/256;
				SendBuf[3]|=0x80;
				SendBuf[4]=Motor_rate%256;
			}
			/* 报尾 */
			SendBuf[5]=0x11;
			
			/* 校验位 */
			SendBuf[6]=SendBuf[0]+SendBuf[1]+SendBuf[2]+SendBuf[3]+SendBuf[4]+SendBuf[5];
			
			FlushInQ (Com_port);
			ComWrt (Com_port, (char*)SendBuf, 7);
			break;
	}
	return 0;
}

int CVICALLBACK SET_PID (int panel, int control, int event,
						 void *callbackData, int eventData1, int eventData2)
{
	float pid_p=0;
	float pid_i=0;
	float pid_d=0;
	int i=0;
	int j=0;
	switch (event)
	{
		case EVENT_COMMIT:
			GetCtrlVal (panelHandle, PANEL_PID_P, &pid_p);
			GetCtrlVal (panelHandle, PANEL_PID_I, &pid_i); 
			GetCtrlVal (panelHandle, PANEL_PID_D, &pid_d); 
			
			pid_p=pid_p*100;
			pid_i=pid_i*100;
			pid_d=pid_d*100;
			
//发送P值			
			/* 报头 */
			SendBuf[0]=0xaa;
			
			/* 主从机 */
			SendBuf[1]=0x01;
			
			/* 命令行 */
			SendBuf[2]=0x03;  //
			
			SendBuf[3]=(unsigned char)pid_p;
			
			SendBuf[4]=0x00;
		
			/* 报尾 */
			SendBuf[5]=0x11;
			
			/* 校验位 */
			SendBuf[6]=SendBuf[0]+SendBuf[1]+SendBuf[2]+SendBuf[3]+SendBuf[4]+SendBuf[5];
			
			FlushOutQ (Com_port);
			ComWrt (Com_port, (char*)SendBuf, 7);
			
			for(i=0;i<10000;i++)
				for(j=0;j<1000;j++);
//发送I值			
			/* 报头 */
			SendBuf[0]=0xaa;
			
			/* 主从机 */
			SendBuf[1]=0x01;
			
			/* 命令行 */
			SendBuf[2]=0x04;  //
			
			SendBuf[3]=(unsigned char)pid_i;
			
			SendBuf[4]=0x00;
		
			/* 报尾 */
			SendBuf[5]=0x11;
			
			/* 校验位 */
			SendBuf[6]=SendBuf[0]+SendBuf[1]+SendBuf[2]+SendBuf[3]+SendBuf[4]+SendBuf[5];
			
			FlushOutQ (Com_port);
			ComWrt (Com_port, (char*)SendBuf, 7);
			
			for(i=0;i<10000;i++)
				for(j=0;j<1000;j++); 
//发送D值
			/* 报头 */
			SendBuf[0]=0xaa;
			
			/* 主从机 */
			SendBuf[1]=0x01;
			
			/* 命令行 */
			SendBuf[2]=0x05;  //
			
			SendBuf[3]=(unsigned char)pid_d;
			
			SendBuf[4]=0x00;
		
			/* 报尾 */
			SendBuf[5]=0x11;
			
			/* 校验位 */
			SendBuf[6]=SendBuf[0]+SendBuf[1]+SendBuf[2]+SendBuf[3]+SendBuf[4]+SendBuf[5];
			
			FlushOutQ (Com_port);
			ComWrt (Com_port, (char*)SendBuf, 7);
			
			break;
	}
	return 0;
}

char DataDeal(void)
{
	char check_bit=0;
	char z=0;
	
	check_bit=ReceiveBuf[0]+ReceiveBuf[1]+ReceiveBuf[2]+ReceiveBuf[3]+ReceiveBuf[4]+ReceiveBuf[5];
	
	if(check_bit==ReceiveBuf[6])//校验正确，数据可以使用        
	{
		if(ReceiveBuf[1]==0x02)  //从机发来的数据
		{
			if(ReceiveBuf[2]==0x01)//开关机命令
			{
				if(ReceiveBuf[3]==0x01)//开机
				{
					SetCtrlVal (panelHandle, PANEL_LED, 1);
				}
				else if(ReceiveBuf[3]==0x00)//关机
				{
					 SetCtrlVal (panelHandle, PANEL_LED, 0);
					 SetCtrlVal (panelHandle, PANEL_MOTOR_SHOW_SPEED, 0);
				}
			}
			if(ReceiveBuf[2]==0x02)//速度命令
			{
				ShowSpeed(ReceiveBuf[3],ReceiveBuf[4]);
			}
		}
		for(z=0;z<10;z++)
		{
			ReceiveBuf[z]=0x00;
		}
		return TRUE;
	}
	else
	{   
		z=0;
		while(ReceiveBuf[z]!=0xaa)
		{
			z++;
			if(z>10)
				break;
		}
		if((ReceiveBuf[z+1]==0x02)&&(z<5))	 //从机
		{
			if(ReceiveBuf[z+2]==0x02)//速度命令
			{
				ShowSpeed(ReceiveBuf[z+3],ReceiveBuf[z+4]);
			}
			
			if(ReceiveBuf[z+2]==0x01)//开关机命令
			{
				if(ReceiveBuf[z+3]==0x01)
				{
					SetCtrlVal (panelHandle, PANEL_LED, 1);
				}
				else
				{
					SetCtrlVal (panelHandle, PANEL_LED, 0);
				}
			}
			return TRUE; 
		}
		for(z=0;z<10;z++)
		{
			ReceiveBuf[z]=0x00;
		}
		return FALSE;
	}
}


char ShowSpeed(unsigned char Buf3,unsigned char Buf4)
{
	
	int motor_speed=0;
	
	if((Buf3&0x80)==0x80)
	{
		motor_speed=(Buf3&0x7f)*256;
		motor_speed=motor_speed+Buf4;
		motor_speed=-motor_speed;
		SetCtrlVal (panelHandle, PANEL_MOTOR_SHOW_SPEED, motor_speed);
//		SetCtrlVal (panelHandle, PANEL_STRIPCHART, motor_speed);
		PlotStripChartPoint(panelHandle, PANEL_STRIPCHART, motor_speed);
	}
	else
	{
		motor_speed=Buf3*256;
		motor_speed=motor_speed+Buf4;
		SetCtrlVal (panelHandle, PANEL_MOTOR_SHOW_SPEED, motor_speed);
//		SetCtrlVal (panelHandle, PANEL_STRIPCHART, motor_speed); 
		PlotStripChartPoint(panelHandle, PANEL_STRIPCHART, motor_speed);
	}
	return TRUE;
}

```

main.h

```c
/**************************************************************************/
/* LabWindows/CVI User Interface Resource (UIR) Include File              */
/*                                                                        */
/* WARNING: Do not add to, delete from, or otherwise modify the contents  */
/*          of this include file.                                         */
/**************************************************************************/

#include <userint.h>

#ifdef __cplusplus
    extern "C" {
#endif

     /* Panels and Controls: */

#define  PANEL                            1
#define  PANEL_COMMANDBUTTON              2       /* control type: command, callback function: quit */
#define  PANEL_UART_NUM                   3       /* control type: ring, callback function: (none) */
#define  PANEL_BAUD_RATE                  4       /* control type: ring, callback function: (none) */
#define  PANEL_CHECK                      5       /* control type: ring, callback function: (none) */
#define  PANEL_DATA_BIT                   6       /* control type: ring, callback function: (none) */
#define  PANEL_STOP_BIT                   7       /* control type: ring, callback function: (none) */
#define  PANEL_START_BUTTON               8       /* control type: command, callback function: START_UART */
#define  PANEL_STOP_BUTTON                9       /* control type: command, callback function: STOP_UART */
#define  PANEL_START_MOTOR                10      /* control type: command, callback function: START_MOTOR */
#define  PANEL_STOP_MOTOR                 11      /* control type: command, callback function: STOP_MOTOR */
#define  PANEL_LED                        12      /* control type: LED, callback function: (none) */
#define  PANEL_SET_SPEED                  13      /* control type: command, callback function: SET_SPEED */
#define  PANEL_MOTOR_SET_RATE             14      /* control type: scale, callback function: (none) */
#define  PANEL_MOTOR_SHOW_SPEED           15      /* control type: scale, callback function: (none) */
#define  PANEL_LED_UART                   16      /* control type: LED, callback function: (none) */
#define  PANEL_LED_2                      17      /* control type: LED, callback function: (none) */
#define  PANEL_PID_P                      18      /* control type: numeric, callback function: (none) */
#define  PANEL_PID_I                      19      /* control type: numeric, callback function: (none) */
#define  PANEL_PID_D                      20      /* control type: numeric, callback function: (none) */
#define  PANEL_SET_PID                    21      /* control type: command, callback function: SET_PID */
#define  PANEL_STRIPCHART                 22      /* control type: strip, callback function: (none) */


     /* Control Arrays: */

          /* (no control arrays in the resource file) */


     /* Menu Bars, Menus, and Menu Items: */

          /* (no menu bars in the resource file) */


     /* Callback Prototypes: */

int  CVICALLBACK quit(int panel, int control, int event, void *callbackData, int eventData1, int eventData2);
int  CVICALLBACK SET_PID(int panel, int control, int event, void *callbackData, int eventData1, int eventData2);
int  CVICALLBACK SET_SPEED(int panel, int control, int event, void *callbackData, int eventData1, int eventData2);
int  CVICALLBACK START_MOTOR(int panel, int control, int event, void *callbackData, int eventData1, int eventData2);
int  CVICALLBACK START_UART(int panel, int control, int event, void *callbackData, int eventData1, int eventData2);
int  CVICALLBACK STOP_MOTOR(int panel, int control, int event, void *callbackData, int eventData1, int eventData2);
int  CVICALLBACK STOP_UART(int panel, int control, int event, void *callbackData, int eventData1, int eventData2);


#ifdef __cplusplus
    }
#endif

```

