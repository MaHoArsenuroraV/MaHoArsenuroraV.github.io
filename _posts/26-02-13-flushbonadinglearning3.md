---
layout: post
title: STM入门学习记录3-OLED显示屏调试工具
categories: [STM32, OLED]
description: STM入门学习记录
keywards: note,record,STM32,OLED
mermaid: true
mathjax: true
---

STM32入门学习记录3：OLED显示屏调试工具

学习自江协科技 [STM32入门教程-2023版 细致讲解 中文字幕](https://www.bilibili.com/video/BV1th411z7sn/?p=4&share_source=copy_web&vd_source=67b9019751b1734c92e834bdee04be24)

# 一、OLED简介
### 1. 简介
OLED（Organic Light Emitting Diode）：有机发光二极管
- OLED显示屏：性能优异的新型显示屏，具有功耗低、响应速度快(高刷新率；总线时序快，避免闭塞程序)、宽视角(从任何视角看显示内容都是清晰的)、轻薄柔韧等特点
- 0.96寸OLED模块：小巧玲珑、占用接口少、简单易用，是电子设计中非常常见的显示屏模块
- 供电：3~5.5V
- 通信协议：I2C/SPI
- 分辨率：128*64

### 2. 硬件电路

![](/images\posts\record\OLED-Hardware-circuit.png)

SCL和SDA为I2C通信引脚，接在单片机I2C通信引脚上，使用GPIO模拟I2C通信则接在GPIO引脚   
7针脚剩下的则是SPI通信引脚

### 3. 驱动函数
![](/images\posts\record\OLED-driving-function.png)
- 参数1为行，参数2为列，显示数字的最后一个参数为长度
- c语言不支持直接写二进制数字
- 若想清除部分字符，则可用显示字符串替换为空格即可
- 使用时将OLED.c中的端口改为所用的

# 二、OLED调试工具
### 1. 本课程内套件OLED可显示内容 
字符、字符串、数字、带符号数字、十六进制/二进制数字

### 2. 常用调试方式
#### 2.1 串口调试
通过串口通信，将调试信息发送到电脑端，电脑使用串口助手显示调试信息
缺陷：<font color=green>通常的串口助手只能通过信息流的方式传递数据，即逐行打印，若需并行检测多个不断变化的数据，则需要**刷屏显示**(不断刷新重写，如while(1))</font>

#### 2.2 显示屏调试
直接将显示屏连接到单片机，将调试信息打印在显示屏上
- 对不断变化的数据，可覆盖刷新显示
- 可做人机交互界面
缺陷：功能有限
#### 2.3 Keil调试模式
借助Keil软件的调试模式，可使用单步运行、设置断点、查看寄存器及变量等功能

#### 2.4 其他调试方法
点灯调试法、注释调试法、对照法....

#### 2.5 总结方法
由此，测试程序的基本思想大致为缩小范围→控制变量→对比测试