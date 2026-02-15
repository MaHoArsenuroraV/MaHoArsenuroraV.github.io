---
layout: post
title: STM入门学习记录4-EXTI外部中断
categories: STM32
description: STM入门学习记录
keywards: note,record,STM32
mermaid: true
mathjax: true
---

STM32入门学习记录4：EXTI外部中断

学习自江协科技 [STM32入门教程-2023版 细致讲解 中文字幕](https://www.bilibili.com/video/BV1th411z7sn/?p=4&share_source=copy_web&vd_source=67b9019751b1734c92e834bdee04be24)

# 一、概念
### 1. 中断
在主程序运行过程中，出现了特定的中断触发条件（中断源），使得CPU暂停当前正在运行的程序，转而去处理中断程序，处理完成后又返回原来被暂停的位置继续运行
### 2. 中断优先级
当有多个中断源同时申请中断时，CPU会根据中断源的轻重缓急进行裁决，优先响应更加紧急的中断源
### 3.中断嵌套
当一个中断程序正在运行时，又有新的更高优先级的中断源申请中断，CPU再次暂停当前中断程序，转而去处理新的中断程序，处理完成后依次进行返回

![](/images\posts\record\Break-execution-process.png)
### 4.STM32中断
f1系列最多68个可屏蔽中断通道，包含：
- EXTI外部中断
- TIM定时器
- ADC模数转换器
- USART串口
- SPI通信
- I2C通信
- RTC实时时钟
- .....

使用NVIC统一管理中断，每个中断通道都拥有16个可编程的优先等级，可对优先级进行分组，进一步设置抢占优先级和响应优先级
![](/images\posts\record\Interrupt-resources-1.png)
![](/images\posts\record\Interrupt-resources-2.png)
![](/images\posts\record\Interrupt-resources-3.png)
![](/images\posts\record\Interrupt-resources-4.png)

- 灰色部分为**内核中断** 其余为**外设中断**
- 中断函数的地址由编译器分配，不固定；而中断跳转由于硬件限制只能跳转至固定地址，为了使硬件跳转至不固定地址的中断函数，则需要在内存中定义一个固定的地址列表，然后在固定的位置由编译器加入跳转到中断函数的代码，该列表即为**中断向量表**

### 5. NVIC(嵌套中断向量控制器)
统一分配中断优先级和管理中断
#### 5.1 NVIC基本结构
![](/images\posts\record\NVIC-basic-structure.png)
#### 5.2 NVIC优先级分组
![](/images\posts\record\NVIC-Priority-Grouping.png)
- **值越小优先级越高**
- 中断号即为上表的'优先级'序号，数值小优先响应

因此，STM32不存在先来后到，而是优先级高先响应

### 6. EXTI(外部中断)
EXTI可以监测指定GPIO口的电平信号，当其指定的GPIO口产生电平变化时，EXTI将立即向NVIC发出中断申请，经过NVIC裁决后即可中断CPU主程序，使CPU执行EXTI对应的中断程序
#### 6.1 简介
- 支持的触发方式：上升沿/下降沿/双边沿/软件触发  
- 支持的GPIO口：所有GPIO口，但相同的Pin不能同时触发中断  
- 通道数：**16个GPIO_Pin(主要通道)**，外加PVD输出、RTC闹钟、USB唤醒、以太网唤醒(蹭网，EXTI有从低功耗停止模式下唤醒STM32的功能)
#### 6.2 触发响应方式：
- 中断响应：
执行中断函数

- 事件响应：
选择触发事件，外部中断信号不通过CPU而是通向其他外设，如ADC转换，DMA触发；属外设间联合操作

#### 6.3 EXTI基本结构
![](/images\posts\record\EXTI-basic-structure.png)
- 经AFIO选择后只有一组GPIO能接入EXTI，故不同GPIO的相同引脚不可能同时接入
- 9-5 15-10分别只占一路资源，即这两组内都只能触发同一个中断函数

#### 6.4 AFIO复用I/O口
AFIO主要用于引脚复用功能的选择和重定义
在STM32中，AFIO主要完成两个任务：复用功能引脚重映射、中断引脚选择

![](/images\posts\record\EXTI-internal-structure.png)
#### 6.5 总结
由以上的特性和介绍可知，外部中断一般用于应对外部驱动的很快的突发信号  
如旋转编码器、**红外遥控接收头**等
- 不推荐使用外部中断读取按键，因为不便消除抖动，可用TIM定时器中断或主程序循环读取

#### 6.6 配件-旋转编码器
用来测量位置、速度或旋转方向的装置，当其旋转轴旋转时，其输出端可以输出与旋转速度和方向对应的方波信号，读取方波信号的频率和相位信息即可得知旋转轴的速度和方向
- 类型：机械触点式/霍尔传感器式/光栅式
非触点式一般可用于电机测速

![](/images\posts\record\Rorary-Encoder-Hardware-circuit.png)
AB则为两相输出

# 二、程序实例
### 1. 对射式红外计数器
#### 1.1 配置GPIO→NVIC联通
1. 配置RCC，将涉及到的外设时钟开启
2. 配置GPIO，选择端口为输入模式
3. 配置AFIO，选择所用GPIO，将其连接到EXTI
4. 配置EXTI，选择边沿触发方式
选择触发响应方式，可选择中断响应和事件响应
5. 配置NVIC，为中断选择合适优先级
- EXTI时钟是打开的 不需要再开启，寄存器中无EXTI时钟控制位
- NVIC是内核外设，所有内核外设的时钟也不需要开启

#### 1.2 AFIO函数
`GPIO_AFIODeInit(void)`  
清除AFIO外设配置  

`GPIO_PinLockConfig(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)`  
锁定GPIO指定引脚配置  

`GPIO_EventOutputConfig(uint8_t GPIO_PortSource, uint8_t GPIO_PinSource)`   
`GPIO_EventOutputCmd(FunctionalState NewState)`   
配置AFIO事件输出功能  

`GPIO_PinRemapConfig(uint32_t GPIO_Remap, FunctionalState NewState)`  
引脚重映射   
两个参数分别为重映射方式和新的状态  
  
`GPIO_EXTILineConfig(uint8_t GPIO_PortSource, uint8_t GPIO_PinSource)`  
配置AFIO的数据选择器，选择想要的中断引脚  

`GPIO_ETH_MediaInterfaceConfig(uint32_t GPIO_ETH_MediaInterface);`  
和以太网有关  

#### 1.3 EXTI函数
`EXTI_DeInit(void)`  
清除EXTI配置  
  
`EXTI_Init(EXTI_InitTypeDef* EXTI_InitStruct)`  
初始化EXTI  
  
`EXTI_StructInit(EXTI_InitTypeDef* EXTI_InitStruct)`  
结构体赋默认值  
  
`EXTI_GenerateSWInterrupt(uint32_t EXTI_Line)`  
参数给一个指定中断线，即可软件触发一次外部中断  
  

`FlagStatus EXTI_GetFlagStatus(uint32_t EXTI_Line);`  
获取指定标志位是否置1    
  
`void EXTI_ClearFlag(uint32_t EXTI_Line);`  
对置1标志位清除  
  
`ITStatus EXTI_GetITStatus(uint32_t EXTI_Line);`  
获取中断标志位是否置1  
  
`void EXTI_ClearITPendingBit(uint32_t EXTI_Line);`  
清除终端挂起标志位  

故前两者适合于在主程序中查看和清除标志位，后两者只能用于中断函数中

也需要定义EXTI的结构体，其有四个成员
- EXTI_Line
  `EXTI_Linex`
- EXTI_LineCmd
  `ENABLE` / `DISABLE`
- EXTI_Mode
  `EXTI_Mode_Event` / `EXTI_Mode_Interrupt`
- EXTI_Trigger
  `EXTI_Trigger_Rising` / `EXTI_Trigger_Falling` / `EXTI_Trigger_Rising_Falling`  

#### 1.4 NVIC函数
`NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup)`  
中断分组，参数为中断分组方式  
  
`NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct)`  
根据结构体初始化NVIC  

`NVIC_SetVectorTable(uint32_t NVIC_VectTab, uint32_t Offset)`  
设置中断向量表  

`NVIC_SystemLPConfig(uint8_t LowPowerMode, FunctionalState NewState)`  
系统低功耗配置  

`SysTick_CLKSourceConfig(uint32_t SysTick_CLKSource)`  
  
  
pre-emption priority and subpriority (抢占优先级和响应优先级)
