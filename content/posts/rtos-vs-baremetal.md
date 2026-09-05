---
title: "裸机开发 vs RTOS：什么时候该上系统？"
date: 2026-08-25T19:00:00+08:00
tags: ["RTOS", "FreeRTOS", "裸机"]
categories: ["嵌入式软件"]
summary: "从一个简单的多任务需求出发，聊聊裸机的 while(1) + 状态机和 RTOS 各自的优缺点。"
---

很多嵌入式项目一开始都是一个大大的 `while(1)` 循环，跑着跑着功能越加越多，就会开始纠结：要不要上 RTOS？

## 裸机开发：简单直接

最原始的写法：

```c
while (1) {
    read_sensor();
    update_display();
    check_button();
    handle_communication();
}
```

问题很快就会暴露：如果 `read_sensor()` 里有一个稍长的延时或者阻塞操作，其他任务就都被卡住了。

进阶一点的做法是用**状态机 + 定时器轮询**，避免阻塞：

```c
typedef enum { STATE_IDLE, STATE_SENSING, STATE_SENDING } AppState;

AppState state = STATE_IDLE;

void app_loop(void) {
    switch (state) {
        case STATE_IDLE:
            if (timer_elapsed(&sensor_timer, 100)) {
                start_sensor_read();
                state = STATE_SENSING;
            }
            break;
        case STATE_SENSING:
            if (sensor_read_done()) {
                state = STATE_SENDING;
            }
            break;
        case STATE_SENDING:
            send_data();
            state = STATE_IDLE;
            break;
    }
}
```

这种写法不阻塞、资源占用极小，很适合任务简单、内存紧张（比如几 KB RAM 的小 MCU）的场景。缺点是任务一多，状态机会变得很难维护，本质上是在裸机上"手写"一个简陋的调度器。

## RTOS：用任务和优先级换可维护性

上了 FreeRTOS 之类的实时系统之后，可以把每个功能拆成独立的任务：

```c
void vSensorTask(void *pvParameters) {
    for (;;) {
        read_sensor();
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}

void vCommTask(void *pvParameters) {
    for (;;) {
        handle_communication();
        vTaskDelay(pdMS_TO_TICKS(10));
    }
}

xTaskCreate(vSensorTask, "Sensor", 128, NULL, 2, NULL);
xTaskCreate(vCommTask, "Comm", 256, NULL, 3, NULL);
```

好处很明显：

- 每个任务逻辑独立，互不阻塞，可读性和可维护性都更好
- 有现成的任务间同步机制：队列、信号量、互斥锁
- 优先级调度让紧急任务（比如通信中断处理）能及时得到 CPU

代价也是实打实的：

- 每个任务都要占用独立的栈空间，RAM 消耗明显增加
- 需要处理好任务间的资源竞争（互斥锁用不好照样死锁）
- 调试比裸机更复杂，任务切换、优先级反转这些问题不容易排查

## 怎么选

个人的经验大致是：

- **RAM 小于几 KB、功能单一、时序要求不复杂**：裸机 + 状态机就够了，没必要上 RTOS。
- **任务多、有明显的并发需求（通信 + 传感器 + 显示 + 按键）、后期还要持续加功能**：上 RTOS 会让代码结构清晰很多，长期维护成本更低。
- **对实时性要求极高的关键路径**（比如电机控制的电流环）：往往还是会用裸机 + 中断，哪怕整体项目跑着 RTOS，关键部分也会尽量减少调度带来的不确定性。

没有绝对的对错，本质上是在"简单可控"和"结构清晰但更复杂"之间做取舍。
