---
title: FreeRTOS索引
author: lixinghui
date: 2023-7-31 12:00:00 +0800
categories: [OS, FreeRTOS]
tags: [OS]
---

> 本文复制于CSDN-安迪西嵌入式，[原文地址](https://andyxi.blog.csdn.net/article/details/115397230)
{: .prompt-tip }



## 可能是最全的 FreeRTOS源码分析及应用开发系列

FreeRTOS 是一个可裁剪的小型且免费的 RTOS 系统，尺寸非常小，可运行于微控制器上。其特点包括：

> – 内核支持抢占式，合作式和时间片调度。
> – 提供了一个用于低功耗的 Tickless 模式。
> – 系统的组件在创建时可以选择动态或者静态的 RAM。
> – FreeRTOS-MPU 支持 Corex-M 系列中的 MPU 单元，如 STM32F429。
> – FreeRTOS 系统简单、小巧、易用，通常情况下内核占用 4k-9k 字节的空间。
> – 高可移植性，代码主要 C 语言编写。
> – 高效的软件定时器，强大的跟踪执行功能，堆栈溢出检测功能。
> – 任务数量不限，任务优先级不限。

本系列通过 23 篇文章详细介绍了 FreeRTOS 的配置与使用，手把手带你分析 FreeRTOS 源码，玩转 FreeRTOS 应用开发

> 下面是各个章节的原文链接地址
{: .prompt-tip }

#### 1. [FreeRTOS 系列 | FreeRTOS 简介](https://blog.csdn.net/Chuangke_Andy/article/details/109597607)

#### 2. [FreeRTOS 系列 | 开发环境](https://blog.csdn.net/Chuangke_Andy/article/details/109672846)

#### 3. [FreeRTOS 系列 | 任务基础知识](https://blog.csdn.net/Chuangke_Andy/article/details/109675031)

#### 4. [FreeRTOS 系列 | 任务创建和删除](https://blog.csdn.net/Chuangke_Andy/article/details/109675112)

#### 5. [FreeRTOS 系列 | 任务挂起和恢复](https://blog.csdn.net/Chuangke_Andy/article/details/109675128)

#### 6. [FreeRTOS 系列 | 多任务调度](https://blog.csdn.net/Chuangke_Andy/article/details/112448391)

#### 7. [FreeRTOS 系列 | 时间管理](https://blog.csdn.net/Chuangke_Andy/article/details/112448467)

#### 8. [FreeRTOS 系列 | 中断管理和临界段](https://blog.csdn.net/Chuangke_Andy/article/details/112506023)

#### 9. [FreeRTOS 系列 | 任务堆栈](https://blog.csdn.net/Chuangke_Andy/article/details/112558512)

#### 10. [FreeRTOS 系列 | 处理器利用率](https://blog.csdn.net/Chuangke_Andy/article/details/112821887)

#### 11. [FreeRTOS 系列 | 任务相关 API 函数](https://blog.csdn.net/Chuangke_Andy/article/details/112825319)

#### 12. [FreeRTOS 系列 | 列表及列表项](https://blog.csdn.net/Chuangke_Andy/article/details/115455482?spm=1001.2014.3001.5501)

#### 13. [FreeRTOS 系列 | 消息队列一](https://blog.csdn.net/Chuangke_Andy/article/details/115488604)

#### 14. [FreeRTOS 系列 | 消息队列二](https://blog.csdn.net/Chuangke_Andy/article/details/115540610?spm=1001.2014.3001.5501)

#### 15. [FreeRTOS 系列 | 二值信号量](https://blog.csdn.net/Chuangke_Andy/article/details/115644441)

#### 16. [FreeRTOS 系列 | 计数信号量](https://blog.csdn.net/Chuangke_Andy/article/details/115700817)

#### 17. [FreeRTOS 系列 | 互斥信号量](https://blog.csdn.net/Chuangke_Andy/article/details/115710230?spm=1001.2014.3001.5501)

#### 18. [FreeRTOS 系列 | 递归互斥信号量](https://blog.csdn.net/Chuangke_Andy/article/details/115741845?spm=1001.2014.3001.5501)

#### 19. [FreeRTOS 系列 | 事件标志组](https://blog.csdn.net/Chuangke_Andy/article/details/115771507?spm=1001.2014.3001.5501)

#### 20. [FreeRTOS 系列 | 软件定时器](https://blog.csdn.net/Chuangke_Andy/article/details/115870112)

#### 21. [FreeRTOS 系列 | 低功耗管理](https://blog.csdn.net/Chuangke_Andy/article/details/115912548)

#### 22. [FreeRTOS 系列 | 内存管理一](https://blog.csdn.net/Chuangke_Andy/article/details/116021779)

#### 23. [FreeRTOS 系列 | 内存管理二](https://blog.csdn.net/Chuangke_Andy/article/details/116085854)



>    以上转载自博主“安迪西嵌入式”的FreeRTOS专栏，仅作学习记录，如有侵权，请联系删除。