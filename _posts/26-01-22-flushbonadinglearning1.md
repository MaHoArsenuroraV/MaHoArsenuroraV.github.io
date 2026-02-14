---
layout: post
title: STM入门学习记录1
categories: STM32
description: STM入门学习记录
keywards: note,record,STM32
mermaid: true
mathjax: true
---

STM32入门学习记录1：新建工程与点灯

学习自江协科技 [STM32入门教程-2023版 细致讲解 中文字幕](https://www.bilibili.com/video/BV1th411z7sn/?p=4&share_source=copy_web&vd_source=67b9019751b1734c92e834bdee04be24)

# 零、常识

### 面包板
内部构造：    
数字为行 字母为列；行内相连，列内隔断，上下为电源正负极，可任取一边/两边使用  
供电行划线处均相连

### stm32开发方式  
- 寄存器
- 标准库
- HAL库

### build & rebuild
build只进行增量编译：  
它会检查文件修改时间，只有当改动过某个 .c 文件，才会重新编译那一个文件，没改过的文件则直接跳过，<font color=blue>速度快</font>  
rebuild则为全量编译：强制把工程里所有的代码文件(包括庞大的库函数)全部重新编译一遍

build使用场景：  
main.c中修改了数行代码  

rebuild使用场景：修改了全局配置或玄学问题，如：  
1. 修改了魔术棒 (Options for Target) 里的设置
2. 修改了头文件
3. 遇到玄学报错
   
### 常见缩写
- **PP (Push-Pull)：推挽**  
既能输出高电平（3.3V），也能输出低电平（0V），驱动能力强
- **OD (Open-Drain)：开漏**  
只能输出低电平（0V），输出高电平时呈“高阻态”（相当于断路）。通常需要外部上拉电阻才能输出高电平
- **AF (Alternate Function)：复用功能**  
引脚的控制权交给片上外设（如串口、SPI、I2C），而不是由 CPU 直接控制高低

### 八种模式(输入输出)
STM32F103有八种模式，其中四种输入四种输出
#### 输入模式
1. **模拟输入** `GPIO_Mode_AIN` (0x0)  
   用途： 使用 ADC（模数转换器）采集引脚电压时使用，关闭引脚上的数字功能（施密特触发器），避免干扰模拟信号  

2. **浮空输入** `GPIO_Mode_IN_FLOATING` (0x04)  
   引脚内部既没有上拉电阻，也没有下拉电阻。如果不接外部信号，引脚电平是不确定的  
   用途： 通信协议（如 I2C 的数据线）、按键检测（外部有电阻时）

3. **下拉输入(Input Pull-Down)** `GPIO_Mode_IPD` (0x28)  
   内部开启一个弱下拉电阻，把电平拉到 GND ;默认状态是低电平  
   用途： 检测高电平有效的信号

4. **上拉输入(Input Pull-Up)** `GPIO_Mode_IPU` (0x48)  
   内部开启一个弱上拉电阻，把电平拉到 VCC ;默认状态是高电平  
   用途： 检测低电平有效的信号（如常见的按键，按下接地）

#### 输出模式
1. **推挽输出** `GPIO_Mode_Out_PP` (0x10)  
输出 1 就是 3.3V，输出 0 就是 0V  
用途： 点亮 LED、驱动蜂鸣器、普通的逻辑控制（最常用）

2. **开漏输出** `GPIO_Mode_Out_OD` (0x14)  
输出 0 接地，输出 1 时引脚悬空  
用途： 
- 电平转换（比如控制一个 5V 的设备，可以外接 5V 上拉电阻）

- “线与”逻辑（多台设备共用一根线，只要有一个拉低，总线就低）

3. **复用推挽输出** `GPIO_Mode_AF_PP` (0x18)  
引脚不由 GPIO_SetBits 控制，而是由外设自动控制  
用途： 
- 串口的 TX 引脚（UART发送）
- SPI 的 MOSI/SCK 引脚
- PWM 波输出（定时器通道）

4. **复用开漏输出** `GPIO_Mode_AF_OD` (0x1C)  
用途： I2C 的 SCL/SDA 引脚（I2C协议要求必须是开漏）  

### 其他问题
stm32引脚上电后，若不初始化，默认是浮空输入

# 一、新建工程

使用keil5进行工程的创建和管理  

-  新建工程时文件夹的重命名很方便 但是工程名的修改可能会导致一些问题，可用通用名project
## 1. 工程构成：
### 1.1 启动部分 (Start)
   - 启动文件 如`startup_stm32f10x_xl.s`  
     视频中所用型号为stm32f103c8t6，启动文件为`startup_stm32f10x_md.s`  
   - 外设寄存器描述文件 `stm32f10x.h`  
      用于描述stm32的寄存器及其地址
   - 时钟配置文件 `system_stm32f10x.c及其头文件`
   - 内核寄存器描述文件 `core_cm3.c及其头文件`  
   
   将以上文件添加入工程的组中，并加入include paths中

### 1.2 工程部分 (User)
   - 工程主文件`main.c`  
      文件最后一行必须为空行 否则会报警报  
      **若想用寄存器开发则可到此为止**
   - `stm32f10x_conf.h`用于配置库函数头文件包含关系，以及用于参数检查的函数定义(为所有库函数所需)
   - `stm32f10x_it.c及其头文件` it(interrupt)，用于存放中断函数
   

### 1.3 库函数 (Library)
- misc文件为内核库函数，其余为内核外的外设库函数  
  <br/>

 做完上述步骤后，在选项-c/c++-define中添加`USE_STDPERIPH_DRIVER`以启用标准库函数，并将User,Library文件夹也加入include paths中  
![](/images\posts\record\260122-1.png)

由此工程建立完毕，只需编辑User组中的文件达成我们的要求

## 2. 基于库函数的工程编写步骤(电灯程序示例)

### 2.1 使能时钟
`RCC_APB2PeriphClockCmd(选择外设,新的状态)`  
- <font color=blue>可以右键，跳转函数定义，查看函数的简介和参数说明</font>(若无法跳转可以build一下让keil生成索引)

### 2.2 配置端口模式
`GPIO_Init(GPIOx(x为A到G，选择配置哪个GPIO),指向GPIO结构体的指针)`  
根据GPIO_Init结构体参数配置  
`GPIO_InitTypeDef GPIO_InitStructure (最好起这种标准的名字)`  
随后可以通过`结构体名.`的结构引出结构体参数  
该结构体包含GPIO_Mode ; GPIO_Speed ; GPIO_Pin 三个参数 具体可去定义查找配置方法  

### 2.3 点灯
`GPIO_SetBits` 置高电平  
`GPIO_ResetBits` 置低电平  
- 板上PC13自带自检灯LED为低电平有效  
