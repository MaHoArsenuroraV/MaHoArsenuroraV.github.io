---
layout: post
title: STM入门学习记录2
categories: [STM32, c语言]
description: STM入门学习记录
keywards: note,record,STM32,c语言
mermaid: true
mathjax: true
---

STM32入门学习记录2：GPIO输出、输入

学习自江协科技 [STM32入门教程-2023版 细致讲解 中文字幕](https://www.bilibili.com/video/BV1th411z7sn/?p=4&share_source=copy_web&vd_source=67b9019751b1734c92e834bdee04be24)

# 一、简介
GPIO，即(General Purpose Input Output) 通用输入输出口，可配置为8种输入输出模式  
引脚电平：0V~3.3V，部分(带FT的)引脚可容忍5V,输出只能最高3.3V(电源为3.3V)  

![](/images\posts\record\STM32F103C8T6-pin-definition.png)  

### 1. 输出模式
- 可控制端口高低电平，故只要是可以用高低电平控制的地方都可以用GPIO来完成，若控制的是大功率设备，只需加入驱动电路即可
- 还可用于模拟通讯协议的输出时序部分，如I2C,SPI等  

### 2. 输入模式
- 可读取端口高低电平或电压，用于按键输入，外接模块(带有数字输出，如光敏、热敏电阻)电平信号输入，(若输出的是模拟量，还可配置为模拟输入模式，配合ADC外设读取端口模拟电压)ADC电压采集，模拟通讯协议接收数据等  
  
### 3. 结构

![](/images\posts\record\GPIO-basic-structure.png)  

如图 寄存器为32位，前十六位对应十六个引脚，后十六位为空，在APB2总线上

![](/images\posts\record\GPIO-pin-structure.png)

#### 上/下拉输入

如图 因引脚悬空时状态不稳定，易受外界扰动，故有上/下拉电阻，上拉即为默认为高电平的输入模式，下拉反之 

#### 输出

输出数据寄存器只能整体读写，若想单独控制某一端口而不影响其他端口，可通过以下两种方式  
- 读出寄存器，用按位与或按位或的方式更改某一位(c语言的`&=` `|=` 与等对应位为1则不变，对应位0则为0，或等反之)再将更改后的数据写回
- (**库函数的实现方式**) 设置位设置/清除寄存器，若想将某位写为1，则将**位设置寄存器**对应位写为1，若想将某位写为0则将**位清除寄存器**对应位写1，写0的位均保持不变
- 读写STM32中的位带位置，stm32中专门分配有一段地址区域映射RAM和外设寄存器的每一位，读写这段地址的数据就相当于读取所映射位置的某一位  

##### 推挽输出
P-MOS和N-MOS均有效，且有强驱动能力，故推挽输出也叫强推模式，数据寄存器为1时，上管导通下管断开，输出高电平；为0则反之，该模式下32对IO口具有绝对控制权

##### 开漏输出
该输出模式仅N-MOS工作    
- 数据寄存器1时，N-MOS断开，高阻模式，可外接上拉电阻5V输出
- 数据寄存器为0，下管导通，接VSS，输出低电平，低电平有驱动能力，高电平没有
- 可作为通讯协议的驱动方式（I2C通讯引脚），多机通信时，该模式可以避免各个设备相互干扰

##### 关闭模式
引脚配置为输入模式时，P-MOS、N-MOS均无效，端口电平由外部信号控制  

### 4. GPIO八大模式
同[前文]({% post_url 26-01-22-flushbonadinglearning1 %})提及

<div style="text-align: center;" markdown="1">

| 模式名称 | 性质 | 特征 |
| :---: | :---: | :---: |
| 浮空输入 | 数字输入 | 可读取引脚电平，若引脚悬空，则电平不确定 |
| 上拉输入 | 数字输入 | 可读取引脚电平，内部连接上拉电阻，悬空时默认高电平 |
| 下拉输入 | 数字输入 | 可读取引脚电平，内部连接下拉电阻，悬空时默认低电平 |
| 模拟输入 | 模拟输入 | GPIO无效，引脚直接接入内部ADC |
| 开漏输出 | 数字输出 | 可输出引脚电平，高电平为高阻态，低电平接VSS |
| 推挽输出 | 数字输出 | 可输出引脚电平，高电平接VDD，低电平接VSS |
| 复用开漏输出 | 数字输出 | 由片上外设控制，高电平为高阻态，低电平接VSS |
| 复用推挽输出 | 数字输出 | 由片上外设控制，高电平接VDD，低电平接VSS |

</div>

- 浮空输入时必须接入连续驱动源，不能出现悬空情况
- 右侧保护二极管上VDD_FT(如图)对容忍5V的接口是特殊的 外部接5v时要特殊处理，否则二极管导通会产生大电流
- 在输出模式下 输入模式也有效，而输入模式下输出无效，因为一个端口只能有一个输出，但是可以有多个输入

![](/images\posts\record\AFPP_AFOD.png)  
  
### 5. GPIO寄存器

**<font color=green>详情可查阅参考手册</font>**
- 端口配置寄存器分高低两个共64位，因为一个端口配置需要四位，具体查阅参考手册
- 端口输入数据寄存器低16位对应16引脚，高16位未使用
- 端口输出数据寄存器同上
- 端口位设置/清除寄存器的高16位为位清除，低16位为位设置
- 端口位清除寄存器低16位同上高16位，方便操作
- 端口配置锁定寄存器：对端口配置锁定防止意外修改

若想对多个端口进行位设置和位清除则用一体的即可，确保同步性

# 二、外部设备电路

### 小功率外设
直接用IO口驱动，注意高低电平

### 大功率外设
IO口控制驱动电路间接控制，如三极管，PNP低电平，NPN反之(发射结正偏)

# 三、程序实例-GPIO输出

### 1. LED闪烁
1. 使能时钟，配置端口模式
2. GPIO输出函数  
   `GPIO_SetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);`  
   将对应端口设为高电平  
   `GPIO_ResetBits(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);`  
   将对应端口设为低电平  
   `GPIO_WriteBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, BitAction BitVal);`  
   前两位仍为GPIO外设指定和端口指定，第三个为bitvalue来设置指定端口`Bit_RESET` `Bit_SET`分别设低/高电平    
   `GPIO_Write(GPIO_TypeDef* GPIOx, uint16_t PortVal);`    
   可同时对16个端口进行写入操作    
   <font color=green>若需要直接写1和0，则需要加上强制类型转换，将1和0类型转换为BitAction的枚举类型`(BitAction)0`</font>
3. 主循环部分  
    使用GPIO输出函数配合Delay函数  

### 2. 跑马灯
1. 使能时钟  
2. 配置所用端口  
   如一次性配置0 1 2三个端口  
   ```
   GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2;
   ```
   因为pin0对应数据为0x0001 pin1为0x0002 pin2为0x0004 pin3为0x0008，即为0001 0010 0100 1000，故可用按位或的方式进行批量选择  
   `GPIO_Pin_ALL`即可选择全部位  
   <font color=green>除此之外，时钟控制也能用按位或的方式操作多个外设，数据的规律多为每一位对应一个外设，GPIO_SetBits等函数也可以同理选择多引脚</font>  
   使用上面的`GPIO_Write(GPIO_TypeDef* GPIOx, uint16_t PortVal)`  
   第二个参数为指定写到输出寄存器的值(直接写到GPIO的ODR寄存器中，故可以写`0x0001`，因为是低电平点亮，所以加按位取反符号`~`，即`~0x0001`)  

### 3. 蜂鸣器
用法同上

# 四、GPIO输入
### 1. 配件
#### 1.1 按键
按下导通，松开断开
**由于内部为机械片，按下会导致几毫秒的机械抖动，故需要进行抖动过滤，否则可能会导致一次按松多次触发，可以设延时解决这一问题**
#### 1.2 传感器模块
光敏/热敏电阻，红外接收管，阻值均与对应信号强度成反比
由于电阻变化不易观察，故一般与定值电阻串联分压，得到模拟电压的输出，从而轻易的检测电压

### 2. 硬件电路

![](/images\posts\record\GPIO-input-hardware-circuit.png)

上方两接法平常是高电平，按下按键转换为低电平；下方两接法相反  
左侧两接法引脚必须为上拉/下拉输入模式；右侧两接法允许引脚浮空模式

### 3. c语言基础
#### 3.1 数据类型

| 关键字 | 位数 | 表示范围 | stdint关键字 | ST关键字 |
| :---: | :---: | :---: | :---: | :---: |
| char | 8 | -128 ~ 127 | int8_t | s8 |
| unsigned char | 8 | 0 ~ 255 | uint8_t | u8 |
| short | 16 | -32768 ~ 32767 | int16_t | s16 |
| unsigned short | 16 | 0 ~ 65535 | uint16_t | u16 |
| int | 32 | -2147483648 ~ 2147483647 | int32_t | s32 |
| unsigned int | 32 | 0 ~ 4294967295 | uint32_t | u32 |
| long | 32 | -2147483648 ~ 2147483647 | | |
| unsigned long | 32 | 0 ~ 4294967295 | | |
| long long | 64 | -(2^63) ~ (2^63)-1 | int64_t | |
| unsigned long long | 64 | 0 ~ (2^64)-1 | uint64_t | |
| float | 32 | -3.4e38 ~ 3.4e38 | | |
| double | 64 | -1.7e308 ~ 1.7e308 | | |

#### 3.2 宏定义
关键字: `#define`
用途：用一个字符串代替一个数字，便于理解，防止出错；提取程序中经常出现参数，便于快速修改  
eg.

```
#define ABC 12345   //定义
int a = ABC         //引用 等效于 int a = 12345
```

#### 3.3 typedef
关键词：`typedef`
用途：将一个长变量类型名换个名字便于使用  
eg.

```
typedef unsigned char unit8_t;  //定义
uint8_t a;                      //引用 等效于unsigned char a;
```
  

- 宏定义不需要`;`，`typedef`必须加
- 宏定义无限制；`typedef`只能给变量类型改名字，对变量类型重命名使用`typedef`更安全

#### 3.4 结构体
关键字：`struct`
用途：数据打包，不同类型变量的集合

```
struct{char x; int y; float z} StructName; //定义
struct{
   char x; 
   int y; 
   float z} StructName;
StructName.x = 'A';                        //引用
StructName.y = 66;
pStructName->z = 1.23;                     //pStructName为结构体地址
```

结构体类型名称较长，可以用`typedef`更改类型名
```
typedef struct{char x; int y; float z} StructName_t;
```

#### 3.5 枚举
关键字：`enum`
用途：定义一个取值受限制的整形变量，用于限制变量取值范围；宏定义的集合
```
enum{FALSE = 0, TRUE = 1} EnumName;  //定义
//花括号内指定变量取值，用`,`隔开，如果等于顺序的值，则可以省略
//可以跟结构体一样换行写
EnumName = FALSE                     //引用 等效EnumName = 0
EnumName = TRUE
```

# 五、程序实例-GPIO输入
### 1.模块化编程——封装驱动程序 (LED)
- LED.h

```
#ifndef __LED_H   //如果没有定义LED字符串
#define __LED_H   //定义LED字符串

void LED_Init(void);
void LED1_ON(void);
void LED1_OFF(void);
void LED2_ON(void);
void LED2_OFF(void);
void LED1_Turn(void);
void LED2_Turn(void);

#endif            //收尾ifndef

```

- LED.c
```
#include "stm32f10x.h"

void LED_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_1 | GPIO_Pin_2;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	GPIO_SetBits(GPIOA, GPIO_Pin_1 | GPIO_Pin_2);
}

void LED1_ON(void)
{
	GPIO_ResetBits(GPIOA, GPIO_Pin_1);
}

void LED1_OFF(void)
{
	GPIO_SetBits(GPIOA, GPIO_Pin_1);
}

void LED1_Turn(void)
{
	if (GPIO_ReadOutputDataBit(GPIOA, GPIO_Pin_1)==0)
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_1);
	}
	else
	{
		GPIO_ResetBits(GPIOA, GPIO_Pin_1);
	}
}

void LED2_ON(void)
{
	GPIO_ResetBits(GPIOA, GPIO_Pin_2);
}

void LED2_OFF(void)
{
	GPIO_SetBits(GPIOA, GPIO_Pin_2);
}

void LED2_Turn(void)
{
	if (GPIO_ReadOutputDataBit(GPIOA, GPIO_Pin_2)==0)
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_2);
	}
	else
	{
		GPIO_ResetBits(GPIOA, GPIO_Pin_2);
	}
}

```

如此便可在`main.c`引用`LED.h`头文件以及`LED_Init()`初始化函数

### 2. 按键控制LED

#### 2.1 函数

`GPIO_ReadInputDataBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)`
读取输入数据寄存器某一端口输入值，参数为端口，返回值为高低电平
`GPIO_ReadInputData(GPIO_TypeDef* GPIOx);`
整个输入数据寄存器
`GPIO_ReadOutputDataBit(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);`
一般用于输出模式下查看输出
`GPIO_ReadOutputData(GPIO_TypeDef* GPIOx);`
读整个输出数据寄存器

#### 2.2 驱动程序封装(按键)
- key.h
```
#ifndef __KEY_H
#define __KEY_H

void Key_Init(void);
uint8_t Key_GetNum(void);
void LED1_Turn(void);
void LED2_Turn(void);

#endif

```
- key.c
```
#include "stm32f10x.h"
#include "Delay.h"


void Key_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_1 | GPIO_Pin_2;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
}

uint8_t Key_GetNum(void)
{
	uint8_t KeyNum = 0;
	if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) ==0)
	{
		Delay_ms(20);
		while (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) ==0)
		Delay_ms(20);
		KeyNum = 1;
	}
	if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_11) ==0)
	{
		Delay_ms(20);
		while (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_11) ==0)
		Delay_ms(20);
		KeyNum = 2;
	}
	return KeyNum;
}

```
#### 2.3 主函数编写
- main.c
```
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "LED.h"
#include "key.h"

uint8_t KeyNum;

int main(void)
{
	LED_Init();
	Key_Init();
	while (1)
	{
		KeyNum = Key_GetNum();
		if (KeyNum == 1)
		{
			LED1_Turn();
		}
		if (KeyNum == 2)
		{
			LED2_Turn();
		}
	}
}

```

### 3.光敏传感器控制蜂鸣器

#### 3.1 封装蜂鸣器驱动文件
蜂鸣器与led同理，光敏传感器只需初始化和返回值函数即可
- LightSensor.c
```
#include "stm32f10x.h"

void LightSensor_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
}

uint8_t LightSensor_Get(void)
{
	return GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_13);
}

```

#### 3.2 主函数编写
- main.c
```
#include "stm32f10x.h"                  // Device header
#include "Buzzer.h"
#include "Delay.h"
#include "LightSensor.h"

uint8_t KeyNum;

int main(void)
{
	Buzzer_Init();
	LightSensor_Init();
	while (1)
	{
		if (LightSensor_Get() == 1)
		{
			Buzzer_ON();
		}
		else
		{
			Buzzer_OFF();
		}
	}
}

```
