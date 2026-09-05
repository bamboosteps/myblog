---
title: "STM32 入门：从点亮一个 LED 开始"
date: 2026-08-10T20:00:00+08:00
tags: ["STM32", "GPIO", "入门"]
categories: ["单片机"]
summary: "几乎每个嵌入式新手的第一个实验：让一个 LED 闪烁起来。这篇笔记记录了用寄存器和 HAL 库两种方式点灯的过程。"
---

学嵌入式和学别的编程语言不太一样，第一个"Hello World"往往不是打印一行字，而是让一颗 LED 闪起来。

## 硬件层面在发生什么

STM32 的每个 GPIO 引脚背后都是一组寄存器，控制着这个引脚的：

- 模式（输入 / 输出 / 复用功能 / 模拟）
- 输出类型（推挽 / 开漏）
- 速度
- 上下拉

点灯本质上就是把某个引脚配置成推挽输出，然后往输出寄存器写 0 或 1。

## 方式一：直接操作寄存器

不用库，直接怼寄存器，能最直观地理解底层原理：

```c
// 使能 GPIOA 时钟
RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;

// 将 PA5 配置为通用输出模式
GPIOA->MODER &= ~(0x3 << (5 * 2));
GPIOA->MODER |=  (0x1 << (5 * 2));

while (1) {
    GPIOA->ODR ^= (1 << 5); // 翻转 PA5
    for (volatile int i = 0; i < 500000; i++); // 简单延时
}
```

这种写法的好处是能清楚地看到每一位寄存器在做什么，坏处是可读性差，换个型号引脚定义就得重写。

## 方式二：使用 HAL 库

实际项目里更常用 HAL（或者 LL 库），可读性好很多：

```c
GPIO_InitTypeDef GPIO_InitStruct = {0};

__HAL_RCC_GPIOA_CLK_ENABLE();

GPIO_InitStruct.Pin = GPIO_PIN_5;
GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
GPIO_InitStruct.Pull = GPIO_NOPULL;
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

while (1) {
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    HAL_Delay(500);
}
```

## 一点心得

- `for` 循环延时只适合最初的实验，正式项目里一定要用定时器或者 `HAL_Delay`（本质也是 SysTick），否则延时会随主频、编译优化等变化而漂移。
- 点灯虽然简单，但涉及到时钟树、GPIO 配置、寄存器操作这几个嵌入式最基础的概念，值得花时间搞明白原理，而不是只会调 HAL 函数。
- 建议在点灯之后，紧接着去看一下自己单片机的**时钟树图**，理解一下 `HAL_Delay` 计时准不准，为什么。

下一篇打算记录一下 UART、I2C、SPI 这几种常见通信协议的区别和使用场景。
