---
title: "STM32H7系列在GCC环境下的ld链接文件配置"
description: "STM32H7B0开发描述"
keywords: "stm32h7,openocd,gcc,链接文件"

date: 2024-08-12T14:41:50+08:00
lastmod: 2024-08-12T14:41:50+08:00

categories:
  - 嵌入式开发
tags:
  - stm32
  - gcc

toc: true
---



> 本文将在 `Clion` + `gcc` + `Openocd` 这一环境下搭建`stm32h7b0`的开发环境。重点涉及 `ld` 文件的修改，以便能够更好地适应H7系列多块RAM分区的利用。在修改`ld`文件的时候，读者应当理解基本的内存模型，并与启动文件`startup.s`联系起来。


STM32h7系列在Clion环境下的搭建

## LD文件基本分析

### 基本的内存模型
在进入LD文件的分析前，我们首先需要知道一些基本的知识：
- 栈和堆的内存增长方向是什么？
- 局部变量、全局变量有什么区别？它们还有别的分类方式吗？或者说你是否知道`BSS`、`Data`这些概念。

我们现在开始分析链接脚本的开头:
```c
/* Entry Point */
ENTRY(Reset_Handler)
```
凭借直觉，我们应当能猜出来开头的`ENTRY(Reset_Handler)`就是代指程序入口，至于`Reset_Handler`，我们知道它在启动文件里。此处无需我们多关心。现在我们接着往下看：

```c
/* Highest address of the user mode stack */
_estack = ORIGIN(DTCMRAM) + LENGTH(DTCMRAM);    /* end of RAM */
/* Generate a link error if heap and stack don't fit into RAM */
_Min_Heap_Size = 0x200;      /* required amount of heap  */
_Min_Stack_Size = 0x400; /* required amount of stack */

/* Specify the memory areas */
MEMORY
{
  ITCMRAM (xrw)  : ORIGIN = 0x00000000, LENGTH = 64K
  FLASH (rx)     : ORIGIN = 0x08000000, LENGTH = 128K
  DTCMRAM (xrw)  : ORIGIN = 0x20000000, LENGTH = 128K
  RAM (xrw)      : ORIGIN = 0x24000000, LENGTH = 1024K
  RAM_CD (xrw)   : ORIGIN = 0x30000000, LENGTH = 128K
  RAM_SRD (xrw)  : ORIGIN = 0x38000000, LENGTH = 32K
}
```

这一段代码里已经包含了不少信息。首先是`MEMORY`字段里分别对应了芯片的编址区域，`ORIGIN` 非常显然是对应区域的起始位置。

## 如何将代码放入ITCM

## DTCM/RAM区域使用