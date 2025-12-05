+++
date = '2025-12-05T20:18:24+08:00'
draft = false
title = '流水灯&按键方向切换'
categories = ["单片机项目"]
+++

---

```c
#include "stm32f10x.h"

// LED引脚定义
#define LED0_PIN     GPIO_Pin_0
#define LED1_PIN     GPIO_Pin_1
#define LED2_PIN     GPIO_Pin_2
#define LED3_PIN     GPIO_Pin_3
#define LED_GPIO     GPIOA
#define LED_RCC      RCC_APB2Periph_GPIOA

// 按键引脚定义
#define KEY0_PIN     GPIO_Pin_0
#define KEY0_GPIO    GPIOC
#define KEY0_RCC     RCC_APB2Periph_GPIOC

// 全局变量
uint8_t currentLed = 0;    // 当前点亮的LED索引
uint8_t direction = 0;     // 方向：0-正向，1-反向

// 延时函数
void Delay(uint32_t nCount)
{
    for(; nCount != 0; nCount--);
}

// GPIO初始化
void LED_GPIO_Config(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;
    
    // 使能LED对应的GPIO时钟
    RCC_APB2PeriphClockCmd(LED_RCC, ENABLE);
    
    // 配置LED引脚为推挽输出
    GPIO_InitStructure.GPIO_Pin = LED0_PIN | LED1_PIN | LED2_PIN | LED3_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(LED_GPIO, &GPIO_InitStructure);
    
    // 初始化所有LED为熄灭状态
    GPIO_ResetBits(LED_GPIO, LED0_PIN);
    GPIO_ResetBits(LED_GPIO, LED1_PIN);
    GPIO_ResetBits(LED_GPIO, LED2_PIN);
    GPIO_ResetBits(LED_GPIO, LED3_PIN);
}

// 按键GPIO初始化
void KEY_GPIO_Config(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;
    
    // 使能按键对应的GPIO时钟
    RCC_APB2PeriphClockCmd(KEY0_RCC, ENABLE);
    
    // 配置按键引脚为上拉输入
    GPIO_InitStructure.GPIO_Pin = KEY0_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_Init(KEY0_GPIO, &GPIO_InitStructure);
}

// 按键扫描函数
uint8_t KEY_Scan(void)
{
    if(GPIO_ReadInputDataBit(KEY0_GPIO, KEY0_PIN) == 0)
    {
        Delay(20000);  // 消抖
        if(GPIO_ReadInputDataBit(KEY0_GPIO, KEY0_PIN) == 0)
        {
            while(GPIO_ReadInputDataBit(KEY0_GPIO, KEY0_PIN) == 0);  // 等待按键释放
            return 1;
        }
    }
    return 0;
}

int main(void)
{
    // 初始化LED和按键GPIO
    LED_GPIO_Config();
    KEY_GPIO_Config();
    
    while (1)
    {
        // 检测按键
        if(KEY_Scan())
        {
            direction = !direction;  // 切换方向
        }
        
        // 先关闭所有LED
        GPIO_ResetBits(LED_GPIO, LED0_PIN);
        GPIO_ResetBits(LED_GPIO, LED1_PIN);
        GPIO_ResetBits(LED_GPIO, LED2_PIN);
        GPIO_ResetBits(LED_GPIO, LED3_PIN);
        
        // 根据方向点亮对应的LED
        if(direction == 0)  // 正向
        {
            switch(currentLed)
            {
                case 0:
                    GPIO_SetBits(LED_GPIO, LED0_PIN);
                    break;
                case 1:
                    GPIO_SetBits(LED_GPIO, LED1_PIN);
                    break;
                case 2:
                    GPIO_SetBits(LED_GPIO, LED2_PIN);
                    break;
                case 3:
                    GPIO_SetBits(LED_GPIO, LED3_PIN);
                    break;
            }
        }
        else  // 反向
        {
            switch(currentLed)
            {
                case 0:
                    GPIO_SetBits(LED_GPIO, LED3_PIN);
                    break;
                case 1:
                    GPIO_SetBits(LED_GPIO, LED2_PIN);
                    break;
                case 2:
                    GPIO_SetBits(LED_GPIO, LED1_PIN);
                    break;
                case 3:
                    GPIO_SetBits(LED_GPIO, LED0_PIN);
                    break;
            }
        }
        
        currentLed = (currentLed + 1) % 4;  // 更新当前LED索引
        Delay(5000000);  // 延时约500ms
    }
}

```

---
