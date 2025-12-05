+++
date = '2025-12-05T20:46:03+08:00'
draft = false
title = 'PWM生成与变换'
categories = ["单片机项目"]
+++

---

```c
#include "stm32f10x.h"

// 函数声明
void RCC_Configuration(void);  // 时钟配置
void GPIO_Configuration(void);  // GPIO配置
void TIM3_Configuration(void); // 定时器3配置

void Delay(__IO uint32_t nCount);

int main(void)
{
    // 初始化系统时钟
    RCC_Configuration();
    // 初始化GPIO
    GPIO_Configuration();
    // 初始化定时器3
    TIM3_Configuration();
    
    // PWM值变量，控制占空比
    uint16_t pwm_value = 0;
    // 方向控制：1-增加PWM值，0-减少PWM值
    uint8_t direction = 1;
    
    // 主循环
    while(1)
    {
        // PWM值变化逻辑
        if(direction)  // 如果方向为增加
        {
            pwm_value += 10;  // PWM值增加10
            if(pwm_value >= 200) direction = 0;  // 达到最大值200后改变方向
        }
        else  // 如果方向为减少
        {
            pwm_value -= 10;  // PWM值减少10
            if(pwm_value <= 0) direction = 1;  // 达到最小值0后改变方向
        }
        
        // 设置TIM3通道2和通道3的比较值，即PWM占空比
        TIM_SetCompare2(TIM3, pwm_value);  // 设置通道2的PWM值
        TIM_SetCompare3(TIM3, pwm_value);  // 设置通道3的PWM值
        
        // 延时函数，控制PWM变化速度
        Delay(0x10000);
    }
}

// 简单延时函数
void Delay(__IO uint32_t nCount)
{
    for(; nCount != 0; nCount--);
}

// 时钟配置函数
void RCC_Configuration(void)
{
    // 使能TIM3时钟，TIM3位于APB1总线上
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
    // 使能GPIOA、GPIOB和AFIO时钟，AFIO用于复用功能
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA | RCC_APB2Periph_GPIOB | RCC_APB2Periph_AFIO, ENABLE);
}

// GPIO配置函数
void GPIO_Configuration(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;
    
    // 配置GPIOB引脚5为复用推挽输出
    // TIM3通道2映射到GPIOB引脚5
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;  // 复用推挽输出模式
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;  // GPIO速度50MHz
    GPIO_Init(GPIOB, &GPIO_InitStructure);
    
    // 配置GPIOB引脚0为复用推挽输出
    // TIM3通道3映射到GPIOB引脚0
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
    GPIO_Init(GPIOB, &GPIO_InitStructure);
}

// 定时器3配置函数
void TIM3_Configuration(void)
{
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;  // 定时器时基结构体
    TIM_OCInitTypeDef TIM_OCInitStructure;  // 定时器输出比较结构体
    
    // 定时器时基配置
    TIM_TimeBaseStructure.TIM_Period = 1000;  // 自动重装载值，决定PWM频率
    // PWM频率 = 72MHz / (71+1) / (1000+1) ≈ 1kHz
    TIM_TimeBaseStructure.TIM_Prescaler = 71;  // 预分频器，72MHz/(71+1)=1MHz
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;  // 向上计数模式
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;  // 时钟分频因子
    TIM_TimeBaseInit(TIM3, &TIM_TimeBaseStructure);  // 初始化定时器3
    
    // PWM模式配置
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;  // PWM模式1
    TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;  // 使能输出
    TIM_OCInitStructure.TIM_Pulse = 500;  // 初始脉冲宽度，即初始占空比为50%
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;  // 输出极性为高
    TIM_OC2Init(TIM3, &TIM_OCInitStructure);  // 初始化TIM3通道2
    
    TIM_OC3Init(TIM3, &TIM_OCInitStructure);  // 初始化TIM3通道3
    
    // 使能预装载寄存器
    TIM_OC2PreloadConfig(TIM3, TIM_OCPreload_Enable);  // 使能通道2预装载
    TIM_OC3PreloadConfig(TIM3, TIM_OCPreload_Enable);  // 使能通道3预装载
    
    // 使能自动重装载预装载
    TIM_ARRPreloadConfig(TIM3, ENABLE);
    
    // 使能定时器3
    TIM_Cmd(TIM3, ENABLE);
}

```

---
