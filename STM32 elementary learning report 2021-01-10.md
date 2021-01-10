@[TOC](STM32初级学习报告二)
# 一．STM32固件库文件基本内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210110182641508.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzUxMzkyMDc2,size_16,color_FFFFFF,t_70)
在这些文件的基础上，我们再新建相关功能文件，编写功能代码，实现相应功能。

# 二．基本函数之GPIO输出输入
## GPIO输出---点亮LED
1. led.h头文件：
①	包含底层封装的寄存器的头文件“stm32f10x.h”;
②	包含能直观表达寄存器含义的宏定义;
③	源文件中函数体的函数声明。
代码如下：

```c
#ifndef __LED_H_
#define __LED_H_

#include "stm32f10x.h"

#define GPIO_INIT_CLK             RCC_APB2Periph_GPIOA
#define GPIO_INIT_PORT            GPIOA
#define GPIO_INIT_PIN             GPIO_Pin_5

#define ON                     1
#define OFF                    0
#define LED(para)         if(para == 1) \
								GPIO_SetBits(GPIO_INIT_PORT, GPIO_INIT_PIN);\
					      else  GPIO_ResetBits(GPIO_INIT_PORT, GPIO_INIT_PIN);

void GPIO_Init_Config(void);

#endif
```
2. led.c源文件---IO口的初始化函数：

```c
#include "led.h"

void GPIO_Init_Config(void)
{
	GPIO_InitTypeDef GPIO_InitStructure;
	
	RCC_APB2PeriphClockCmd(GPIO_INIT_CLK, ENABLE);

	GPIO_InitStructure.GPIO_Pin = GPIO_INIT_PIN;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIO_INIT_PORT, &GPIO_InitStructure);
}
```
3. main.c文件---具体功能实现：

```c
#include "stm32f10x.h"
#include "led.h"

void delay(uint32_t count)
{
	for( ; count != 0;count--);
}

int main(void)
{
	GPIO_Init_Config();
	while(1)
	{
		LED(ON);
		//GPIO_SetBits(GPIO_INIT_PORT, GPIO_INIT_PIN);
		delay(0XFFFFF);
		LED(OFF);
		//GPIO_ResetBits(GPIO_INIT_PORT, GPIO_INIT_PIN);
		delay(0XFFFFF);
	}
}
```
## GPIO输入---按键检测
只需在点亮LED的基础上增加：
1. key.h头文件:

```c
#ifndef __KEY_H_
#define __KEY_H_

#include "stm32f10x.h"

#define KEY_INIT_CLK             RCC_APB2Periph_GPIOC
#define KEY_INIT_PORT            GPIOC
#define KEY_INIT_PIN             GPIO_Pin_13

#define KEY_ON                   0
#define KEY_OFF					 1

void KEY_Init_Config(void);
void Key_Scan(GPIO_TypeDef* GPIOx,uint16_t GPIO_PIN);

#endif
```
2. key.c源文件：

```c
#include "key.h"
#include "led.h"

void KEY_Init_Config(void)
{
	GPIO_InitTypeDef GPIO_InitStructure;
	
	RCC_APB2PeriphClockCmd(KEY_INIT_CLK, ENABLE);

	GPIO_InitStructure.GPIO_Pin = KEY_INIT_PIN;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
	GPIO_Init(KEY_INIT_PORT, &GPIO_InitStructure);
}

void Key_Scan(GPIO_TypeDef* GPIOx,uint16_t GPIO_PIN)
{
	if(GPIO_ReadInputDataBit(GPIOx, GPIO_PIN) == KEY_ON)
		LED_TOGGLE;
}
```

3. main.c文件:

```c
#include "stm32f10x.h"
#include "led.h"
#include "key.h"

void delay(uint32_t count)
{
	for( ; count != 0;count--);
}

int main(void)
{
	GPIO_Init_Config();
	KEY_Init_Config();
	
	while(1)
	{
		#if 0
		LED(ON);
		//GPIO_SetBits(GPIO_INIT_PORT, GPIO_INIT_PIN);
		delay(0XFFFFF);
		LED(OFF);
		//GPIO_ResetBits(GPIO_INIT_PORT, GPIO_INIT_PIN);
		delay(0XFFFFF);
		
		#elif 1
		Key_Scan(KEY_INIT_PORT, KEY_INIT_PIN);
		
		#endif
	}
}
```
## 位带操作---记住公式就好了，也不经常用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210110183920458.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzUxMzkyMDc2,size_16,color_FFFFFF,t_70)
# 三.复位和时钟控制（RCC）
## 1.时钟树
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210110184054186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzUxMzkyMDc2,size_16,color_FFFFFF,t_70)
## 2.系统时钟的配置
    三种不同的时钟源可被用来驱动系统时钟（SYSCLK）：
    ◾HSI振荡器时钟
    ◾HSE振荡器时钟
    ◾PLL时钟
其中 HSE和HIS时钟源 可以直接作为系统时钟来源；也可以通过PLL(锁相环(PhaseLockedLoop))间接作为系统时钟来源。
HSE可选择不分频或二分频后作为PLL时钟来源，HIS必须二分频后作为PLL时钟来源；再乘以所选PLL倍频系数得到PLL时钟源作为系统时钟。（一般选择HSE = 8MHz，不分频，作为PLL时钟来源，经9倍频后得到PLL时钟作为系统时钟源）
注：系统时钟的配置都是通过操作RCC相应寄存器的相应位的写入或读取值来实现的。固件库已将这部分代码实现，无需我们再自行配置。但在有特殊需求的情况下，我们也可以根据STM32参考手册RCC寄存器描述更改系统时钟来源和系统时钟大小。









