---
title: 单片机软件框架
author: lixinghui
date: 2023-4-12 12:00:00 +0800
categories: [OS, User]
tags: [OS]
---




## 三大模式

单片机的软件设计大致分为三类，分别为：

-   前后台顺序执行模式
-   时间片轮询模式
-   实时操作系统模式

## 前后台顺序执行模式

对于很多需求简单的软件，它的代码简单，或者有些单片机性能较差的芯片，前后台模式的设计是最友好的，它不需要考虑整体的实时性和并发性，对于外设初始化之后就只要在`while(1)`里面循环执行用户代码就可以了。

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
    MX_TIM7_Init();
    MX_USART1_UART_Init();
    /* USER CODE BEGIN 2 */
    HAL_TIM_Base_Start_IT(&htim7);
    /* USER CODE END 2 */

    /* Infinite loop */
    /* USER CODE BEGIN WHILE */
    while (1)
    {
        /* USER CODE END WHILE */

        /* USER CODE BEGIN 3 */
        if (task1_flag)
        {
            task1_flag = 0;
            HAL_GPIO_TogglePin(LED0_GPIO_Port, LED0_Pin);
        }
    }
    /* USER CODE END 3 */
}

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    static uint32_t tick = 0;
    if (htim == &htim7)
    {
        tick++;
        tick % 1000 == 0 ? (task1_flag = 1, tick = 0) : 0;
    }
}
```



## 时间片轮询模式

在一些任务需要频繁进入同时等待的时候，如按键扫描等，就可以安排上时间片轮询，通过建立时间片，对每个任务分配一个时间片，到达时间片就进入任务内进行执行，在执行完成之后，就退出时间片，这时候CPU就切换到其他任务去执行。

需要注意的是，时间片轮询模式需要一个定时器，时间片不能设计的太短，不然会一直卡死。其他任务无法执行，需要按照实际需求进行时间片的设计。

### 基本设计模式

```c
typedef struct
{
    uint32_t tick;
    uint8_t ms10 : 1;
    uint8_t ms50 : 1;
    uint8_t ms100 : 1;
    uint8_t ms200 : 1;
    uint8_t ms500 : 1;
    uint8_t sec1 : 1;
} TIM_FLAG;

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
    MX_TIM7_Init();
    MX_USART1_UART_Init();
    /* USER CODE BEGIN 2 */
    HAL_TIM_Base_Start_IT(&htim7);
    /* USER CODE END 2 */

    /* Infinite loop */
    /* USER CODE BEGIN WHILE */
    while (1)
    {
        /* USER CODE END WHILE */

        /* USER CODE BEGIN 3 */
        if (tim_flag.ms100)
        {
            tim_flag.ms100 = 0;
            HAL_GPIO_TogglePin(LED0_GPIO_Port, LED0_Pin);
        }

        if (tim_flag.ms500)
        {
            tim_flag.ms500 = 0;
            HAL_GPIO_TogglePin(LED1_GPIO_Port, LED1_Pin);
        }

        if (tim_flag.sec1)
        {
            tim_flag.sec1 = 0;
            printf("one second pass\r\n");
            // HAL_UART_Transmit(&huart1, "one second pass\r\n", 17, 0xffff);
        }
    }
    /* USER CODE END 3 */
}

/*因为HAL库执行效率较低，需要快速响应的地方可以替换HAL库的代码*/
/**
 * @brief This function handles TIM7 global interrupt.
 */
void TIM7_IRQHandler(void)
{
    /* USER CODE BEGIN TIM7_IRQn 0 */

    /* USER CODE END TIM7_IRQn 0 */
    // HAL_TIM_IRQHandler(&htim7);
    /* USER CODE BEGIN TIM7_IRQn 1 */
    UserTIM_IRQHandler(&htim7);
    /* USER CODE END TIM7_IRQn 1 */
}

void UserTIM_IRQHandler(TIM_HandleTypeDef *htim)
{
    if (htim == &htim7)
    {
        __HAL_TIM_CLEAR_FLAG(htim, TIM_FLAG_UPDATE);
        tim_flag.tick++;
        tim_flag.tick % 10 == 0 ? tim_flag.ms10 = 1 : 0;
        tim_flag.tick % 50 == 0 ? tim_flag.ms50 = 1 : 0;
        tim_flag.tick % 100 == 0 ? tim_flag.ms100 = 1 : 0;
        tim_flag.tick % 200 == 0 ? tim_flag.ms200 = 1 : 0;
        tim_flag.tick % 500 == 0 ? tim_flag.ms500 = 1 : 0;
        tim_flag.tick % 1000 == 0 ? (tim_flag.sec1 = 1, tim_flag.tick = 0) : 0;
    }
}


```

### 带函数指针的设计方式

```c
typedef struct
{
    uint8_t run_flag;        /*运行标志位*/
    uint16_t tick;           /*计数器*/
    uint16_t period;         /*周期*/
    void (*pTaskHook)(void); /*任务函数*/
} TaskInfoType;

/* USER CODE BEGIN PFP */
void LED0_Task(void);
void LED1_Task(void);
void UART_Task(void);
void TASK_Remark(void);
void TASK_Process(void);
/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
TaskInfoType task_info[MAX_TASKS] = {
    {0, 100, 100, LED0_Task},
    {0, 500, 500, LED1_Task},
    {0, 1000, 1000, UART_Task}};
/* USER CODE END 0 */


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
    MX_TIM7_Init();
    MX_USART1_UART_Init();
    /* USER CODE BEGIN 2 */
    HAL_TIM_Base_Start_IT(&htim7);
    /* USER CODE END 2 */

    /* Infinite loop */
    /* USER CODE BEGIN WHILE */
    while (1)
    {
        /* USER CODE END WHILE */

        /* USER CODE BEGIN 3 */
        TASK_Process();
    }
    /* USER CODE END 3 */
}


void LED0_Task(void)
{
    HAL_GPIO_TogglePin(LED0_GPIO_Port, LED0_Pin);
}

void LED1_Task(void)
{
    HAL_GPIO_TogglePin(LED1_GPIO_Port, LED1_Pin);
}

void UART_Task(void)
{
    printf("USART Task running\r\n");
}


/**
 * @brief 中断回调函数中执行
 *
 */
void TASK_Remark(void)
{
    uint8_t i;
    for (i = 0; i < MAX_TASKS; i++)
    {
        if (task_info[i].tick)
        {
            task_info[i].tick--;
            if (0 == task_info[i].tick)
            {
                task_info[i].tick = task_info[i].period;
                task_info[i].run_flag = 1;
            }
        }
    }
}

/**
 * @brief 主函数while(1)中执行
 *
 */
void TASK_Process(void)
{
    uint8_t i;
    for (i = 0; i < MAX_TASKS; i++)
    {
        if (task_info[i].run_flag)
        {
            task_info[i].run_flag = 0; /*清空标志位*/
            task_info[i].pTaskHook();  /*执行函数*/
        }
    }
}

void UserTIM_IRQHandler(TIM_HandleTypeDef *htim)
{
    if (htim == &htim7)
    {
        __HAL_TIM_CLEAR_FLAG(htim, TIM_FLAG_UPDATE);
        TASK_Remark();
    }
}
```

## 实时操作系统

嵌入式操作系统有很多，不包括linux，适合于单片机上运行的嵌入式操作系统（EOS）最常用的有FreeRTOS\UCOS\RT-Thread。

![](https://pic1.zhimg.com/70/v2-2999b8fdc5f069d6c94ed9c14d22f5e0_1440w.avis?source=172ae18b&biz_tag=Post)


参考：

[嵌入式软件开发常用的三种架构你知道吗？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/504173751)

