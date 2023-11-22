---
title: 合作式调度器
author: lixinghui
date: 2023-4-13 12:00:00 +0800
categories: [OS, User]
tags: [OS]
---




在[调度器简要说明](https://annie-timor.github.io/2023/04/13/单片机软件框架_2/)中讲了调度器的分类及比较，同时应该根据不同的场景需求来选择合适的调度器，在这一篇博客主要讲解合作式调度器的具体设计。

## 基本设计模式

```c
#ifndef _SCHEDULER_H_
#define _SCHEDULER_H_

#include "main.h"
#include <stdint.h>

typedef struct
{
    void (*task_function)(void); /*任务函数*/
    uint32_t period;             /*任务周期ms*/
    uint32_t delay;              /*任务延迟ms*/
    uint32_t tick;               /*任务时钟ms*/
    uint8_t run_flag;            /*任务运行*/
} TASK_HANDLE;

void SCH_Update(void);
uint8_t SCH_AddTask(void (*task_function)(), int period, int delay);
void SCH_Start(void);
void SCH_DispatchTask(void);
#endif // !_SCHEDULER_H_

```
```c
#include "scheduler.h"
#include "tim.h"
#include <stdio.h>
#include <stdlib.h>

#define CAN_USE_MALLOC 0
#define TASK_MAX 3 // maximum number of tasks

TASK_HANDLE *task[TASK_MAX] = {NULL}; // task handle point

#if !CAN_USE_MALLOC
TASK_HANDLE task_info[TASK_MAX] = {0};
#endif

/**
 * @brief 调度器更新，由中断服务函数调用
 *
 */
void SCH_Update(void)
{
    uint8_t i = 0;
    /*清空中断标志位*/
    __HAL_TIM_CLEAR_IT(&htim6, TIM_IT_UPDATE);

    for (i = 0; i < TASK_MAX; i++)
    {
        /*检测是否有任务*/
        if (task[i] != NULL)
        {
            /*先走完延迟*/
            task[i]->delay == 0 ? task[i]->tick-- : task[i]->delay--;

            if (0 == task[i]->tick)
            {
                /*任务需要运行*/
                task[i]->run_flag += 1;
                task[i]->tick = task[i]->period;
            }
        }
    }
}

/**
 * @brief 调度器添加任务
 *
 * @param task_function 任务函数
 * @param period 任务周期
 * @param delay 任务延迟
 * @return uint8_t 成功/失败
 */
uint8_t SCH_AddTask(void (*task_function)(), int period, int delay)
{
    uint8_t i;
    /*寻找一个空的指针*/
    for (i = 0; i < TASK_MAX; i++)
    {
        if (task[i] == NULL)
        {
            break;
        }
    }
    if (i == TASK_MAX)
    {
        /*无空指针*/
        return 0;
    }
#if CAN_USE_MALLOC
    task[i] = (TASK_HANDLE *)malloc(sizeof(TASK_HANDLE));
    if (task[i] == NULL)
    {
        /*内存分配失败*/
        return 0;
    }
#else
    task[i] = &task_info[i];
#endif

    task[i]->task_function = task_function;
    task[i]->period = period;
    task[i]->delay = delay;
    task[i]->tick = task[i]->period;
    task[i]->run_flag = 0;

    return 1;
}

/**
 * @brief 开启调度器
 *
 */
void SCH_Start(void)
{
    HAL_TIM_Base_Start_IT(&htim6);
}

/**
 * @brief 调度器迅速处理任务
 *
 */
void SCH_DispatchTask(void)
{
    uint8_t i = 0;
    for (i = 0; i < TASK_MAX; i++)
    {
        if (task[i]->run_flag > 0)
        {
            task[i]->task_function();
            task[i]->run_flag -= 1;
        }
    }
}

```
```c
/**
 * @brief  The application entry point.
 * @retval int
 */
int main(void)
{
    /* USER CODE BEGIN 1 */

    /* USER CODE END 1 */

    /* MCU Configuration--------------------------------------------------------*/

    /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
    HAL_Init();

    /* USER CODE BEGIN Init */

    /* USER CODE END Init */

    /* Configure the system clock */
    SystemClock_Config();

    /* USER CODE BEGIN SysInit */

    /* USER CODE END SysInit */

    /* Initialize all configured peripherals */
    MX_GPIO_Init();
    MX_TIM6_Init();
    MX_USART1_UART_Init();
    /* USER CODE BEGIN 2 */

    /*添加任务*/
    SCH_AddTask(Task_LED0, 500, 0);
    SCH_AddTask(Task_LED1, 250, 100);
    SCH_AddTask(Task_Printf, 1000, 500);

    /*打开调度器*/
    // HAL_TIM_Base_Start_IT(&htim6);
    SCH_Start();

    /* USER CODE END 2 */

    /* Infinite loop */
    /* USER CODE BEGIN WHILE */
    while (1)
    {
        /* USER CODE END WHILE */

        /* USER CODE BEGIN 3 */
        /*开始任务调度*/
        SCH_DispatchTask();
    }
    /* USER CODE END 3 */
}

/* USER CODE BEGIN 4 */
void Task_LED0(void)
{
    HAL_GPIO_TogglePin(LED0_GPIO_Port, LED0_Pin);
}

void Task_LED1(void)
{
    HAL_GPIO_TogglePin(LED1_GPIO_Port, LED1_Pin);
}

void Task_Printf(void)
{
    printf("one second already\r\n");
}
/* USER CODE END 4 */

```

## 加入任务状态，运行队列，优先级

```c
#ifndef _SCHEDULER_H_
#define _SCHEDULER_H_

#include "main.h"

#define SCH_ERROR 0
#define SCH_OK 1

#define CPU_USAGE /*CPU使用率测量*/

typedef enum
{
    SUSPEND = 0, /*挂起态*/
    RUNNING = 1, /*运行态*/
} TaskState;

typedef struct
{
    void (*task_function)(void); /*任务函数*/
    uint32_t period;             /*任务周期*/
    uint8_t priority;            /*任务优先级*/
    uint32_t tick;               /*任务时钟*/
    uint32_t delay;              /*延迟*/
    TaskState state;             /*任务状态*/
} TaskHandle;

typedef struct TaskNode
{
    struct TaskNode *prev; /*指向上一个任务*/
    TaskHandle data;       /*节点数据*/
    struct TaskNode *next; /*指向下一个任务*/
} TaskNode;

uint8_t SCH_Init(void);
void SCH_Start(void);
void SCH_Update(void);
uint8_t SCH_AddTask(void (*task_function)(), int period, char priority, int delay);
uint8_t SCH_DeleteTask(void (*task_function)());
uint8_t SCH_SuspendTask(void (*task_function)());
uint8_t SCH_ResumeTask(void (*task_function)());
uint16_t SCH_TaskUsage(void);
void SCH_DispatchTask(void);
#endif // !_SCHEDULER_H_

```

```c
#include "scheduler.h"
#include "queue.h"
#include "tim.h"
#include <stdio.h>
#include <stdlib.h>

TaskNode *task_head = NULL;

#ifdef CPU_USAGE
uint8_t cpu_state = 0;  /*cpu状态*/
uint32_t task_time = 0; /*工作时间*/
uint32_t idle_time = 0; /*空闲时间*/
#endif                  // CPU_USAGE

/**
 * @brief 调度器初始化
 *
 * @return uint8_t
 */
uint8_t SCH_Init(void)
{
    task_head = calloc(1, sizeof(TaskNode));
    if (task_head == NULL)
    {
        /*内存分配失败*/
        return SCH_ERROR;
    }
    task_head->prev = NULL;
    task_head->next = NULL;

    return SCH_OK;
}

/**
 * @brief 开启调度器
 *
 */
void SCH_Start(void)
{
    HAL_TIM_Base_Start_IT(&htim6);
}

/**
 * @brief 调度器更新，在中断服务函数中执行
 *
 */
void SCH_Update(void)
{
    TaskNode *current = task_head;
    /*清空中断标志位*/
    __HAL_TIM_CLEAR_IT(&htim6, TIM_IT_UPDATE);
#ifdef CPU_USAGE
    if (cpu_state)
    {
        task_time++;
    }
    else
    {
        idle_time++;
    }
    if ((task_time > 0xffffff) || (idle_time > 0xffffff))
    {
        task_time >>= 16;
        idle_time >>= 16;
    }
#endif
    current = task_head;
    do
    {
        current = current->next; // task_head->next为第一个任务
        if (current->data.state == RUNNING)
        {
            /*先走完延迟*/
            current->data.delay == 0 ? current->data.tick-- : current->data.delay--;
            if (0 == current->data.tick)
            {
                /*任务需要运行*/
                QueuePush(current->data.task_function, current->data.priority);
                current->data.tick = current->data.period;
            }
        }
    } while (current->next != NULL);
}

/**
 * @brief 添加任务
 *
 * @param task_function 任务回调函数
 * @param period 周期
 * @param priority 优先级
 * @param delay 延迟
 * @return uint8_t
 */
uint8_t SCH_AddTask(void (*task_function)(), int period, char priority, int delay)
{
    TaskNode *current = task_head;

    // 获取最后一个指针的地址
    for (current = task_head; current->next != NULL; current = current->next)
        ;

    // 建立新节点，处理指针
    TaskNode *newNode = calloc(1, sizeof(TaskNode));
    if (newNode == NULL)
    {
        /*内存分配失败*/
        return SCH_ERROR;
    }

    newNode->data.task_function = task_function;
    newNode->data.period = period;
    newNode->data.priority = priority;
    newNode->data.tick = period;
    newNode->data.delay = delay;
    newNode->data.state = RUNNING;
    /*建立连接*/
    newNode->prev = current;
    newNode->next = NULL;
    current->next = newNode;

    return SCH_OK;
}

/**
 * @brief 删除任务
 *
 * @param task_function 任务执行函数
 * @return uint8_t
 */
uint8_t SCH_DeleteTask(void (*task_function)())
{
    TaskNode *current = NULL;
    /*获取要删除的任务或者最后*/
    current = task_head;
    do
    {
        current = current->next;
    } while ((current->data.task_function != task_function) && (current->next != NULL));

    if (current->data.task_function != task_function)
    {
        /*可用任务内没找到要删除的任务*/
        return SCH_ERROR;
    }

    current->prev->next = current->next;
    if (current->next != NULL)
        current->next->prev = current->prev;

    free(current);
    current = NULL;
    return SCH_OK;
}

/**
 * @brief 挂起任务
 *
 * @param task_function 任务执行函数
 * @return uint8_t
 */
uint8_t SCH_SuspendTask(void (*task_function)())
{
    TaskNode *current = NULL;
    /*获取要删除的任务或者最后*/
    current = task_head;
    do
    {
        current = current->next;
    } while ((current->data.task_function != task_function) && (current->next != NULL));

    if (current->data.task_function != task_function)
    {
        /*可用任务内没找到要删除的任务*/
        return SCH_ERROR;
    }

    current->data.state = SUSPEND;

    return SCH_OK;
}

/**
 * @brief 恢复任务的执行
 *
 * @param task_function 任务执行函数
 * @return uint8_t
 */
uint8_t SCH_ResumeTask(void (*task_function)())
{
    TaskNode *current = NULL;
    /*获取要删除的任务或者最后*/
    current = task_head;
    do
    {
        current = current->next;
    } while ((current->data.task_function != task_function) && (current->next != NULL));

    if (current->data.task_function != task_function)
    {
        /*可用任务内没找到要删除的任务*/
        return SCH_ERROR;
    }

    current->data.state = RUNNING;

    return SCH_OK;
}

#ifdef CPU_USAGE
/**
 * @brief 返回CPU的使用率
 *
 * @return uint16_t
 */
uint16_t SCH_TaskUsage(void)
{
    return ((task_time * 100) / (task_time + idle_time));
}
#endif

/**
 * @brief 快速执行需要运行的任务
 *
 */
void SCH_DispatchTask(void)
{
    uint8_t ret = 0;
    void (*task_function)(void);

    ret = QueuePop(&task_function);
    if (ret != QUEUE_EMPTY)
    {
#ifdef CPU_USAGE
        cpu_state = 1;
#endif // CPU_USAGE
        (*task_function)();
#ifdef CPU_USAGE
        cpu_state = 0;
#endif // CPU_USAGE
    }
}

```

```c
#ifndef _QUEUE_H
#define _QUEUE_H

#define QUEUE_SIZE 32 /*队列大小*/

#include "main.h"

typedef enum
{
    QUEUE_EMPTY = 0,
    QUEUE_FULL = 1,
    QUEUE_OK = 2,
} QueueState;

typedef struct
{
    uint16_t head;                /*队列头*/
    uint16_t tail;                /*队列尾*/
    uint16_t size;                /*队列大小*/
    void (*data[QUEUE_SIZE])();   /*队列内数据*/
    uint8_t priority[QUEUE_SIZE]; /*队列优先级*/
} Queue_t;

void QueueInit(void);
uint8_t QueuePush(void (*data)(void), uint8_t priority);
uint8_t QueuePop(void (*(*data))(void));
#endif // !_QUEUE_H

```

```c
#include "queue.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static Queue_t sch_queue = {NULL};

/**
 * @brief 队列初始化
 *
 */
void QueueInit(void)
{
    Queue_t *q = &sch_queue;
    q->head = 0;
    q->tail = 0;
    q->size = 0;

    for (uint8_t i = 0; i < QUEUE_SIZE; i++)
    {
        q->data[i] = NULL;
        q->priority[i] = 0;
    }
}

/**
 * @brief 任务进队
 *
 * @param data 任务调用的函数
 * @param priority 优先级
 * @return uint8_t
 */
uint8_t QueuePush(void (*data)(void), uint8_t priority)
{
    uint8_t i, j;
    Queue_t *q = &sch_queue;
    /*判断队列是否已满*/
    if (((q->head % QUEUE_SIZE) == q->tail) && (q->size == QUEUE_SIZE))
    {
        return QUEUE_FULL;
    }

    /*找到入队的位置*/
    /*tail优先级最高--head最低*/
    for (i = q->tail;; i = (i + 1) % QUEUE_SIZE)
    {
        if (priority > q->priority[i])
        {
            break;
        }
        else
        {
            if (i == (q->head + 1) % QUEUE_SIZE)
            {
                /*循环一周依旧没找到*/
                break;
            }
        }
    }

    if (i != (q->head + 1) % QUEUE_SIZE)
    {
        /*队列后移*/
        for (j = q->head;; j = (j - 1) % QUEUE_SIZE)
        {
            q->data[(j + 1) % QUEUE_SIZE] = q->data[j];
            q->priority[(j + 1) % QUEUE_SIZE] = q->priority[j];
            if (j == i)
            {
                break;
            }
            j == 0 ? j = QUEUE_SIZE : 0;
        }
    }

    /*存入数据进队列*/
    q->data[i] = data;
    q->priority[i] = priority;
    /*队头后移*/
    q->head = (q->head + 1) % QUEUE_SIZE;
    q->size++;
    return QUEUE_OK;
}

/**
 * @brief 出队，弹出优先级最高的任务
 *
 * @return uint8_t
 */
uint8_t QueuePop(void (*(*data))(void))
{
    Queue_t *q = &sch_queue;
    /*判断队列是否为空*/
    if ((q->head == q->tail) && (q->size == 0))
    {
        return QUEUE_EMPTY;
    }
    /*读取数据*/
    *data = q->data[q->tail];
    /*清零已使用数据*/
    q->data[q->tail] = NULL;
    q->priority[q->tail] = 0;
    /*队尾前移*/
    q->tail = (q->tail + 1) % QUEUE_SIZE;
    q->size--;

    return QUEUE_OK;
}

```

```c
/**
 * @brief  The application entry point.
 * @retval int
 */
int main(void)
{
    /* USER CODE BEGIN 1 */

    /* USER CODE END 1 */

    /* MCU Configuration--------------------------------------------------------*/

    /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
    HAL_Init();

    /* USER CODE BEGIN Init */

    /* USER CODE END Init */

    /* Configure the system clock */
    SystemClock_Config();

    /* USER CODE BEGIN SysInit */

    /* USER CODE END SysInit */

    /* Initialize all configured peripherals */
    MX_GPIO_Init();
    MX_TIM6_Init();
    MX_USART1_UART_Init();
    /* USER CODE BEGIN 2 */

    /*队列初始化*/
    QueueInit();
    /*调度器初始化*/
    SCH_Init();

    /*添加任务*/
    SCH_AddTask(Task_LED0, 500, 1, 50);
    SCH_AddTask(Task_LED1, 250, 2, 10);
    SCH_AddTask(Task_Printf, 1000, 3, 200);
    SCH_AddTask(Task_KeyScan, 10, 4, 5);

    /*打开调度器*/
    SCH_Start();

    /* USER CODE END 2 */

    /* Infinite loop */
    /* USER CODE BEGIN WHILE */
    while (1)
    {
        /* USER CODE END WHILE */

        /* USER CODE BEGIN 3 */
        /*开始任务调度*/
        SCH_DispatchTask();
    }
    /* USER CODE END 3 */
}



/* USER CODE BEGIN 4 */
void Task_LED0(void)
{
    HAL_GPIO_TogglePin(LED0_GPIO_Port, LED0_Pin);
}

void Task_LED1(void)
{
    float in1 = 0.0f;
    float ans = 0.0f;

    for (short i = 0; i < 5000; i++)
    {
        in1 += 1.0f;
        ans = sqrt(in1);
    }

    HAL_GPIO_TogglePin(LED1_GPIO_Port, LED1_Pin);
}

void Task_Printf(void)
{
    printf("one second already\r\n");
}

void Task_KeyScan(void)
{
    static uint8_t task_state = RUNNING;
    uint8_t key_value = 0;
    key_value = KeyScan();

    if (key_value == KEY1_SHORT_PRES)
    {
        printf("CPU Usage:%d\r\n", SCH_TaskUsage());

        if (task_state == RUNNING)
        {
            if (SCH_OK == SCH_SuspendTask(Task_Printf))
            {
                printf("Suspend Task Printf\r\n");
                task_state = SUSPEND;
            }
        }
        else if (task_state == SUSPEND)
        {
            if (SCH_OK == SCH_ResumeTask(Task_Printf))
            {
                printf("Resume Task Printf\r\n");
                task_state = RUNNING;
            }
        }
    }
}
```

## 混合式调度器

混合式调度器是基于合作式调度器的，它将部分任务的优先级设置到最大，在中断回调函数中，在判断函数是否需要执行时，如果检测到是需要立即执行的函数，也就是优先级最大的函数，就不再将该函数进入队列中，而是直接在中断服务函数中就执行完成。

设计如下：

```c
#define MAX_PRIORITY 0xff

/*初始化的时候将函数优先级设置为最大*/
SCH_AddTask(Task_Printf, 1000, MAX_PRIORITY, 200);


/**
 * @brief 调度器更新，在中断服务函数中执行
 *
 */
void SCH_Update(void)
{
    TaskNode *current = task_head;
    /*清空中断标志位*/
    __HAL_TIM_CLEAR_IT(&htim6, TIM_IT_UPDATE);
#ifdef CPU_USAGE
    if (cpu_state)
    {
        task_time++;
    }
    else
    {
        idle_time++;
    }
    if ((task_time > 0xffffff) || (idle_time > 0xffffff))
    {
        task_time >>= 16;
        idle_time >>= 16;
    }
#endif
    current = task_head;
    do
    {
        current = current->next; // task_head->next为第一个任务
        if (current->data.state == RUNNING)
        {
            /*先走完延迟*/
            current->data.delay == 0 ? current->data.tick-- : current->data.delay--;
            if (0 == current->data.tick)
            {
                /*任务需要运行*/
                if (MAX_PRIORITY == current->data.priority)
                {
                	/*直接执行，不放入队列，保证及时性*/
                    (*current->data.task_function)();
                }
                else
                {
                    QueuePush(current->data.task_function, current->data.priority);
                }
                current->data.tick = current->data.period;
            }
        }
    } while (current->next != NULL);
}
```

