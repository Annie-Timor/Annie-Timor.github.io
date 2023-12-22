---
title: FreeRTOS内存分配
author: lixinghui
date: 2023-11-30 9:10:00 +0800
categories: [mcu]
tags: [嵌入式杂谈]
---

FreeRTOS使用pvPortMalloc()函数来替代malloc()申请内存，使用vPortFree()函数来替代 free()释放内存

不同的嵌入式系统对于内存分配和时间要求不同，内存分配算法可作为系统的可选选项，FreeRTOS使用者就可以使用适合的内存分配方法。FreeRTOS提供了5中内存分配方法，这5种方法分别对应heap_1.c、heap_2.c、heap_3.c、heap_4.c、heap_5.c这五个文件。下面将会详见介绍这五种方法的区别

分配方法	区别
heap_1.c	最简单，但是只能申请内存，不能释放
heap_2.c	提供了内存释放函数，但是不能合并内存块，易导致内存碎片
heap_3.c	对标准C中的malloc()和free()的简单封装，并提供了线程保护
heap_4.c	在heap_2.c的基础上增加了内存块合并功能，降低了内存碎片的产生
heap_5.c	在heap_4.c的基础上，支持内存堆使用不连续的内存块

这里仅针对heap_1和heap_4做记录

---

## heap1

heap_1内存分配的特点如下：

>   -   适用于创建好任务、信号量和队列就不会删除的应用
>   -   具有可确定性，不会导致内存碎片
>   -   代码实现和内存分配过程简单，内存是从一个静态数字中分配，适合于不需要动态内存分配的应用

### heap1.h

```c
#ifndef _HEAP1_H
#define _HEAP1_H
#include <stdint.h>

void *pvPortMalloc(size_t xWantedSize);
void vPortFree(void *pv);
void vPortInitialiseBlocks(void);
size_t xPortGetFreeHeapSize(void);

#endif
```



### heap1.c

```c

#include <stdint.h>
#include <stdlib.h>
#include <string.h>
#include "heap1.h"

/*每个字节8位*/
#define heapBITS_PER_BYTE ((size_t)8)

/*8字节对齐方式*/
#define portBYTE_ALIGNMENT 8

#if portBYTE_ALIGNMENT == 8 /*掩蔽掉低7位的字节*/
#define portBYTE_ALIGNMENT_MASK (0x0007)
#elif portBYTE_ALIGNMENT == 4 /*掩蔽掉低3位的字节*/
#define portBYTE_ALIGNMENT_MASK (0x0003)
#endif

/*配置总堆区大小*/
#define configTOTAL_HEAP_SIZE (1024)
/* 对齐堆起始地址的字节可能会丢失几个字节。 */
#define configADJUSTED_HEAP_SIZE (configTOTAL_HEAP_SIZE - portBYTE_ALIGNMENT)

#define portPOINTER_SIZE_TYPE (int *)

/* 模拟时没有用上的函数 */
#define vTaskSuspendAll()
#define xTaskResumeAll()
#define mtCOVERAGE_TEST_MARKER()
#define traceMALLOC(pvAddress, uiSize)
#define traceFREE(pvAddress, uiSize)

/*断言*/
#define configASSERT(x) \
    if ((x) == 0)       \
    {                   \
        for (;;)        \
            ;           \
    }

/* Allocate the memory for the heap. */
#if (configAPPLICATION_ALLOCATED_HEAP == 1)
// 需要用户自行定义内存堆
extern uint8_t ucHeap[configTOTAL_HEAP_SIZE];
#else
static uint8_t ucHeap[configTOTAL_HEAP_SIZE];
#endif /* configAPPLICATION_ALLOCATED_HEAP */

static size_t xNextFreeByte = (size_t)0;
static uint8_t *pucAlignedHeap = NULL;

/*-----------------------------------------------------------*/

void *pvPortMalloc(size_t xWantedSize)
{
    void *pvReturn = NULL;

/* 确保块始终与所需的字节数对齐。 */
#if (portBYTE_ALIGNMENT != 1)
    {
        if (xWantedSize & portBYTE_ALIGNMENT_MASK)
        {
            /* 字节对齐 */
            xWantedSize += (portBYTE_ALIGNMENT - (xWantedSize & portBYTE_ALIGNMENT_MASK));
        }
    }
#endif

    vTaskSuspendAll();//挂起任务调度器
    {
        if (pucAlignedHeap == NULL)
        {
            /* 起始地址做8字节对齐 */
            // pucAlignedHeap = (uint8_t *)(((size_t)&ucHeap[portBYTE_ALIGNMENT]) & (~(portBYTE_ALIGNMENT_MASK)));
            pucAlignedHeap = (uint8_t *)(((size_t)&ucHeap + (portBYTE_ALIGNMENT - 1)) & (~(portBYTE_ALIGNMENT_MASK)));
        }

        /* 检查是否有足够的内存供分配，以及是否越界 */
        if (((xNextFreeByte + xWantedSize) < configADJUSTED_HEAP_SIZE) &&
            ((xNextFreeByte + xWantedSize) > xNextFreeByte)) 
        {
            /*返回申请到的内存首地址*/
            pvReturn = pucAlignedHeap + xNextFreeByte;
            xNextFreeByte += xWantedSize;
        }

        traceMALLOC(pvReturn, xWantedSize);
    }
    xTaskResumeAll();//恢复任务调度器

    return pvReturn;
}
/*-----------------------------------------------------------*/

void vPortFree(void *pv)
{
    (void)pv;

    /* Force an assert as it is invalid to call this function. */
    // configASSERT(pv == NULL);
}
/*-----------------------------------------------------------*/

void vPortInitialiseBlocks(void)
{
    /* Only required when static memory is not cleared. */
    xNextFreeByte = (size_t)0;
    pucAlignedHeap = NULL;
    memset(ucHeap, 0, configTOTAL_HEAP_SIZE);
}
/*-----------------------------------------------------------*/

size_t xPortGetFreeHeapSize(void)
{
    return (configADJUSTED_HEAP_SIZE - xNextFreeByte);
}
```



---

## heap4

heap_4提供了一个最优的匹配算法，与heap_2不同，heap_4会将内存碎片合并成一个大的可用内存块

heap_4内存分配的特点如下：

-   可用在需要重复创建和删除任务、队列、信号量等的应用中
-   不会像heap_2那样产生严重的内存碎片
-   具有不确定性，但比标准C中的malloc()和free()效率高



### heap4.h

```c
#ifndef _HEAP4_H
#define _HEAP4_H
#include <stdint.h>

void *pvPortMalloc(size_t xWantedSize);
void vPortFree(void *pv);
size_t xPortGetFreeHeapSize(void);
size_t xPortGetMinimumEverFreeHeapSize(void);


#endif
```



### heap4.c

```c
#include <stdint.h>
#include <stdlib.h>

#include "heap4.h"

/* Block sizes must not get too small. */ /*最小块大小*/
#define heapMINIMUM_BLOCK_SIZE ((size_t)(xHeapStructSize * 2))

/* Assumes 8bit bytes! */ /*每个字节8位*/
#define heapBITS_PER_BYTE ((size_t)8)

/*8字节对齐方式*/
#define portBYTE_ALIGNMENT 8

/*掩蔽掉低7位的字节*/
#if portBYTE_ALIGNMENT == 8
#define portBYTE_ALIGNMENT_MASK (0x0007)
#endif

/*配置总堆区大小*/
#define configTOTAL_HEAP_SIZE (1024)

/* 没有用上的函数 */
#define vTaskSuspendAll()
#define xTaskResumeAll()
#define mtCOVERAGE_TEST_MARKER()
#define traceMALLOC(pvAddress, uiSize)
#define traceFREE(pvAddress, uiSize)

/*断言*/
#define configASSERT(x) \
    if ((x) == 0)       \
    {                   \
        for (;;)        \
            ;           \
    }

/* Allocate the memory for the heap. */
#if (configAPPLICATION_ALLOCATED_HEAP == 1)
/* The application writer has already defined the array used for the RTOS
heap - probably so it can be placed in a special segment or address. */
extern uint8_t ucHeap[configTOTAL_HEAP_SIZE];
#else
static uint8_t ucHeap[configTOTAL_HEAP_SIZE] __attribute__((aligned(8)));
#endif /* configAPPLICATION_ALLOCATED_HEAP */

/* Define the linked list structure.  This is used to link free blocks in order
of their memory address. */
typedef struct A_BLOCK_LINK
{
    struct A_BLOCK_LINK *pxNextFreeBlock; /*<< The next free block in the list. */
    size_t xBlockSize;                    /*<< The size of the free block. */
} BlockLink_t;

/*-----------------------------------------------------------*/

/*
 * Inserts a block of memory that is being freed into the correct position in
 * the list of free memory blocks.  The block being freed will be merged with
 * the block in front it and/or the block behind it if the memory blocks are
 * adjacent to each other.
 */
static void prvInsertBlockIntoFreeList(BlockLink_t *pxBlockToInsert);

/*
 * Called automatically to setup the required heap structures the first time
 * pvPortMalloc() is called.
 */
static void prvHeapInit(void);

/*-----------------------------------------------------------*/

/* The size of the structure placed at the beginning of each allocated memory
block must by correctly byte aligned. */
static const size_t xHeapStructSize = ((sizeof(BlockLink_t) + (((size_t)portBYTE_ALIGNMENT_MASK) - (size_t)1)) & ~((size_t)portBYTE_ALIGNMENT_MASK));

/* Create a couple of list links to mark the start and end of the list. */
static BlockLink_t xStart, *pxEnd = NULL;

/* Keeps track of the number of free bytes remaining, but says nothing about
fragmentation. */
static size_t xFreeBytesRemaining = 0U;            /*空闲块剩余*/
static size_t xMinimumEverFreeBytesRemaining = 0U; /*历史最小空闲块大小*/

/* Gets set to the top bit of an size_t type.  When this bit in the xBlockSize
member of an BlockLink_t structure is set then the block belongs to the
application.  When the bit is free the block is still part of the free heap
space. */
static size_t xBlockAllocatedBit = 0;

/*-----------------------------------------------------------*/

void *pvPortMalloc(size_t xWantedSize)
{
    BlockLink_t *pxBlock, *pxPreviousBlock, *pxNewBlockLink;
    void *pvReturn = NULL;

    vTaskSuspendAll();
    {
        /* If this is the first call to malloc then the heap will require
        initialisation to setup the list of free blocks. */
        if (pxEnd == NULL) /*第一次调用进行初始化*/
        {
            prvHeapInit();
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }

        /* Check the requested block size is not so large that the top bit is
        set.  The top bit of the block size member of the BlockLink_t structure
        is used to determine who owns the block - the application or the
        kernel, so it must be free. */
        if ((xWantedSize & xBlockAllocatedBit) == 0)
        {
            /* The wanted size is increased so it can contain a BlockLink_t
            structure in addition to the requested amount of bytes. */
            if (xWantedSize > 0)
            {
                xWantedSize += xHeapStructSize; /*加上结构体的大小*/

                /* Ensure that blocks are always aligned to the required number
                of bytes. */
                if ((xWantedSize & portBYTE_ALIGNMENT_MASK) != 0x00)
                {
                    /* Byte alignment required. */
                    xWantedSize += (portBYTE_ALIGNMENT - (xWantedSize & portBYTE_ALIGNMENT_MASK));
                    configASSERT((xWantedSize & portBYTE_ALIGNMENT_MASK) == 0);
                }
                else
                {
                    mtCOVERAGE_TEST_MARKER();
                }
            }
            else
            {
                mtCOVERAGE_TEST_MARKER();
            }

            if ((xWantedSize > 0) && (xWantedSize <= xFreeBytesRemaining))
            {
                /* Traverse the list from the start	(lowest address) block until
                one	of adequate size is found. */
                pxPreviousBlock = &xStart;
                pxBlock = xStart.pxNextFreeBlock;
                while ((pxBlock->xBlockSize < xWantedSize) && (pxBlock->pxNextFreeBlock != NULL))
                {
                    pxPreviousBlock = pxBlock;
                    pxBlock = pxBlock->pxNextFreeBlock;
                }

                /* If the end marker was reached then a block of adequate size
                was	not found. */
                if (pxBlock != pxEnd)
                {
                    /* Return the memory space pointed to - jumping over the
                    BlockLink_t structure at its start. */
                    pvReturn = (void *)(((uint8_t *)pxPreviousBlock->pxNextFreeBlock) + xHeapStructSize);
                        /*   = (void *)((uint8_t *)pxBlock + xHeapStructSize); */            

                    /* This block is being returned for use so must be taken out
                    of the list of free blocks. */
                    pxPreviousBlock->pxNextFreeBlock = pxBlock->pxNextFreeBlock;

                    /* If the block is larger than required it can be split into
                    two. */
                    if ((pxBlock->xBlockSize - xWantedSize) > heapMINIMUM_BLOCK_SIZE)
                    {
                        /* This block is to be split into two.  Create a new
                        block following the number of bytes requested. The void
                        cast is used to prevent byte alignment warnings from the
                        compiler. */
                        pxNewBlockLink = (void *)(((uint8_t *)pxBlock) + xWantedSize);
                        configASSERT((((uint32_t)pxNewBlockLink) & portBYTE_ALIGNMENT_MASK) == 0);

                        /* Calculate the sizes of two blocks split from the
                        single block. */
                        pxNewBlockLink->xBlockSize = pxBlock->xBlockSize - xWantedSize;
                        pxBlock->xBlockSize = xWantedSize;

                        /* Insert the new block into the list of free blocks. */
                        prvInsertBlockIntoFreeList((pxNewBlockLink));
                    }
                    else
                    {
                        mtCOVERAGE_TEST_MARKER();
                    }

                    xFreeBytesRemaining -= pxBlock->xBlockSize;

                    if (xFreeBytesRemaining < xMinimumEverFreeBytesRemaining)
                    {
                        xMinimumEverFreeBytesRemaining = xFreeBytesRemaining;
                    }
                    else
                    {
                        mtCOVERAGE_TEST_MARKER();
                    }

                    /* The block is being returned - it is allocated and owned
                    by the application and has no "next" block. */
                    pxBlock->xBlockSize |= xBlockAllocatedBit;
                    pxBlock->pxNextFreeBlock = NULL;
                }
                else
                {
                    mtCOVERAGE_TEST_MARKER();
                }
            }
            else
            {
                mtCOVERAGE_TEST_MARKER();
            }
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }

        traceMALLOC(pvReturn, xWantedSize);
    }
    xTaskResumeAll();

#if (configUSE_MALLOC_FAILED_HOOK == 1)
    {
        if (pvReturn == NULL)
        {
            extern void vApplicationMallocFailedHook(void);
            vApplicationMallocFailedHook();
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }
    }
#endif

    configASSERT((((uint32_t)pvReturn) & portBYTE_ALIGNMENT_MASK) == 0);
    return pvReturn;
}
/*-----------------------------------------------------------*/

void vPortFree(void *pv)
{
    uint8_t *puc = (uint8_t *)pv;
    BlockLink_t *pxLink;

    if (pv != NULL)
    {
        /* The memory being freed will have an BlockLink_t structure immediately
        before it. */
        puc -= xHeapStructSize;

        /* This casting is to keep the compiler from issuing warnings. */
        pxLink = (void *)puc;

        /* Check the block is actually allocated. */
        configASSERT((pxLink->xBlockSize & xBlockAllocatedBit) != 0);
        configASSERT(pxLink->pxNextFreeBlock == NULL);

        if ((pxLink->xBlockSize & xBlockAllocatedBit) != 0)
        {
            if (pxLink->pxNextFreeBlock == NULL)
            {
                /* The block is being returned to the heap - it is no longer
                allocated. */
                pxLink->xBlockSize &= ~xBlockAllocatedBit;

                vTaskSuspendAll();
                {
                    /* Add this block to the list of free blocks. */
                    xFreeBytesRemaining += pxLink->xBlockSize;
                    traceFREE(pv, pxLink->xBlockSize);
                    prvInsertBlockIntoFreeList(((BlockLink_t *)pxLink));
                }
                xTaskResumeAll();
            }
            else
            {
                mtCOVERAGE_TEST_MARKER();
            }
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }
    }
}
/*-----------------------------------------------------------*/

size_t xPortGetFreeHeapSize(void)
{
    return xFreeBytesRemaining;
}
/*-----------------------------------------------------------*/

size_t xPortGetMinimumEverFreeHeapSize(void)
{
    return xMinimumEverFreeBytesRemaining;
}
/*-----------------------------------------------------------*/

void vPortInitialiseBlocks(void)
{
    /* This just exists to keep the linker quiet. */
}
/*-----------------------------------------------------------*/

static void prvHeapInit(void)
{
    BlockLink_t *pxFirstFreeBlock; /*第一个空的块*/
    uint8_t *pucAlignedHeap;
    uint32_t ulAddress;                            /*地址*/
    size_t xTotalHeapSize = configTOTAL_HEAP_SIZE; /*总的堆大小*/

    /* Ensure the heap starts on a correctly aligned boundary. */
    ulAddress = (uint32_t)ucHeap; /*获取堆的起始地址*/

    /*进行8字节对齐获取堆的起始地址，重新计算总的堆大小*/
    if ((ulAddress & portBYTE_ALIGNMENT_MASK) != 0)
    {
        ulAddress += (portBYTE_ALIGNMENT - 1);
        ulAddress &= ~((uint32_t)portBYTE_ALIGNMENT_MASK);
        xTotalHeapSize -= ulAddress - (uint32_t)ucHeap;
    }

    /*变量转地址*/
    pucAlignedHeap = (uint8_t *)ulAddress;

    /*获取堆的起始和结束地址*/
    /* xStart is used to hold a pointer to the first item in the list of free
    blocks.  The void cast is used to prevent compiler warnings. */
    xStart.pxNextFreeBlock = (void *)pucAlignedHeap;
    xStart.xBlockSize = (size_t)0;

    /* pxEnd is used to mark the end of the list of free blocks and is inserted
    at the end of the heap space. */
    ulAddress = ((uint32_t)pucAlignedHeap) + xTotalHeapSize;
    ulAddress -= xHeapStructSize; /*减去一个结构体大小*/
    ulAddress &= ~((uint32_t)portBYTE_ALIGNMENT_MASK);
    pxEnd = (void *)ulAddress;
    pxEnd->xBlockSize = 0;
    pxEnd->pxNextFreeBlock = NULL;

    /*获取第一个空闲块（堆区开始）的起始地址和大小，位置在堆的起始处*/
    /* To start with there is a single free block that is sized to take up the
    entire heap space, minus the space taken by pxEnd. */
    pxFirstFreeBlock = (void *)pucAlignedHeap;
    pxFirstFreeBlock->xBlockSize = ulAddress - (uint32_t)pxFirstFreeBlock;
    pxFirstFreeBlock->pxNextFreeBlock = pxEnd;

    /* Only one block exists - and it covers the entire usable heap space. */
    xMinimumEverFreeBytesRemaining = pxFirstFreeBlock->xBlockSize;
    xFreeBytesRemaining = pxFirstFreeBlock->xBlockSize;

    /* Work out the position of the top bit in a size_t variable. */
    xBlockAllocatedBit = ((size_t)1) << ((sizeof(size_t) * heapBITS_PER_BYTE) - 1);
}
/*-----------------------------------------------------------*/

static void prvInsertBlockIntoFreeList(BlockLink_t *pxBlockToInsert)
{
    BlockLink_t *pxIterator;
    uint8_t *puc;

    /* Iterate through the list until a block is found that has a higher address
    than the block being inserted. */
    for (pxIterator = &xStart; pxIterator->pxNextFreeBlock < pxBlockToInsert; pxIterator = pxIterator->pxNextFreeBlock)
    {
        /* Nothing to do here, just iterate to the right position. */
    }

    /* Do the block being inserted, and the block it is being inserted after
    make a contiguous block of memory? */
    puc = (uint8_t *)pxIterator;
    if ((puc + pxIterator->xBlockSize) == (uint8_t *)pxBlockToInsert)
    {
        pxIterator->xBlockSize += pxBlockToInsert->xBlockSize;
        pxBlockToInsert = pxIterator;
    }
    else
    {
        mtCOVERAGE_TEST_MARKER();
    }

    /* Do the block being inserted, and the block it is being inserted before
    make a contiguous block of memory? */
    puc = (uint8_t *)pxBlockToInsert;
    if ((puc + pxBlockToInsert->xBlockSize) == (uint8_t *)pxIterator->pxNextFreeBlock)
    {
        if (pxIterator->pxNextFreeBlock != pxEnd)
        {
            /* Form one big block from the two blocks. */
            pxBlockToInsert->xBlockSize += pxIterator->pxNextFreeBlock->xBlockSize;
            pxBlockToInsert->pxNextFreeBlock = pxIterator->pxNextFreeBlock->pxNextFreeBlock;
        }
        else
        {
            pxBlockToInsert->pxNextFreeBlock = pxEnd;
        }
    }
    else
    {
        pxBlockToInsert->pxNextFreeBlock = pxIterator->pxNextFreeBlock;
    }

    /* If the block being inserted plugged a gab, so was merged with the block
    before and the block after, then it's pxNextFreeBlock pointer will have
    already been set, and should not be set here as that would make it point
    to itself. */
    if (pxIterator != pxBlockToInsert)
    {
        pxIterator->pxNextFreeBlock = pxBlockToInsert;
    }
    else
    {
        mtCOVERAGE_TEST_MARKER();
    }
}
```

