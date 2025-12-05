+++
date = '2025-12-05T18:43:12+08:00'
draft = false
title = 'LCD代码 Lcd1602 Driver'
categories = ["单片机项目"]
+++

---

main.c
```c
#include "stm32f10x.h"
#include "LCD1602.h"
#include "stdio.h"
#define DELAY_TIME 500000

void Delay(__IO uint32_t nCount)
{
  while(nCount--);
}

int main(void)
{
    // 初始化LCD1602
    LCD_Init();
    
    // 显示字符串
    LCD_Clear();
    LCD_ShowString(1, 1, "STM32F103R6");
    LCD_ShowString(2, 1, "LCD1602 Test");
    
    Delay(DELAY_TIME);
    
    // 显示数字
    LCD_Clear();
    LCD_ShowString(1, 1, "Number: ");
    LCD_ShowNum(1, 9, 12345, 5);
    
    Delay(DELAY_TIME);
    
    // 显示十六进制数
    LCD_Clear();
    LCD_ShowString(1, 1, "Hex: 0x");
    LCD_ShowHexNum(1, 7, 43981, 4);
    
    Delay(DELAY_TIME);
    
    // 显示二进制数
    LCD_Clear();
    LCD_ShowString(1, 1, "Bin: ");
    LCD_ShowBinNum(1, 6, 170, 8);
    
    Delay(DELAY_TIME);
    
    // 显示有符号数
    LCD_Clear();
    LCD_ShowString(1, 1, "Signed: ");
    LCD_ShowSignedNum(1, 9, -123, 4);
    
    Delay(DELAY_TIME);
    
    while (1)
    {
        // 循环显示
        LCD_Clear();
        LCD_ShowString(1, 1, "STM32F103R6");
        LCD_ShowString(2, 1, "LCD1602 Test");
        
        Delay(DELAY_TIME);
        
        LCD_Clear();
        LCD_ShowString(1, 1, "Hello World!");
        LCD_ShowString(2, 1, "MCU Project");
        
        Delay(DELAY_TIME);
    }
}
```

lcd1602.h
```c
#ifndef __LCD1602_H__
#define __LCD1602_H__

#include "stm32f10x.h"

#define LCD_RS_PIN      GPIO_Pin_8
#define LCD_RS_GPIO     GPIOC
#define LCD_RS_RCC      RCC_APB2Periph_GPIOC

#define LCD_RW_PIN      GPIO_Pin_9
#define LCD_RW_GPIO     GPIOC
#define LCD_RW_RCC      RCC_APB2Periph_GPIOC

#define LCD_EN_PIN      GPIO_Pin_10
#define LCD_EN_GPIO     GPIOC
#define LCD_EN_RCC      RCC_APB2Periph_GPIOC

#define LCD_DATA_GPIO   GPIOC
#define LCD_DATA_RCC    RCC_APB2Periph_GPIOC
#define LCD_DATA_PINS   (GPIO_Pin_0|GPIO_Pin_1|GPIO_Pin_2|GPIO_Pin_3|GPIO_Pin_4|GPIO_Pin_5|GPIO_Pin_6|GPIO_Pin_7)

void LCD_Init(void);
void LCD_Clear(void);
void LCD_ShowChar(unsigned char Line, unsigned char Column, char Char);
void LCD_ShowString(unsigned char Line, unsigned char Column, char *String);
void LCD_ShowNum(unsigned char Line, unsigned char Column, unsigned int Number, unsigned char Length);
void LCD_ShowSignedNum(unsigned char Line, unsigned char Column, int Number, unsigned char Length);
void LCD_ShowHexNum(unsigned char Line, unsigned char Column, unsigned int Number, unsigned char Length);
void LCD_ShowBinNum(unsigned char Line, unsigned char Column, unsigned int Number, unsigned char Length);

#endif
```

lcd1602.c
```c
#include "stm32f10x.h"
#include "LCD1602.h"

//私有函数声明
static void LCD_GPIO_Config(void);
static void LCD_Delay_ms(uint32_t ms);
static void LCD_WriteCommand(unsigned char Command);
static void LCD_WriteData(unsigned char Data);
static void LCD_SetCursor(unsigned char Line, unsigned char Column);

/**
  * @brief  LCD1602延时函数，1ms
  * @param  ms 延时毫秒数
  * @retval 无
  */
static void LCD_Delay_ms(uint32_t ms)
{
    uint32_t i;
    for(; ms > 0; ms--)
        for(i = 0; i < 7200; i++);
}

/**
  * @brief  LCD1602写命令
  * @param  Command 要写入的命令
  * @retval 无
  */
static void LCD_WriteCommand(unsigned char Command)
{
    GPIO_ResetBits(LCD_RS_GPIO, LCD_RS_PIN);  // RS = 0，写命令
    GPIO_ResetBits(LCD_RW_GPIO, LCD_RW_PIN);  // RW = 0，写操作
    
    // 写入命令数据
    LCD_DATA_GPIO->ODR = (LCD_DATA_GPIO->ODR & 0xFF00) | Command;
    
    GPIO_SetBits(LCD_EN_GPIO, LCD_EN_PIN);    // EN = 1
    LCD_Delay_ms(1);
    GPIO_ResetBits(LCD_EN_GPIO, LCD_EN_PIN);  // EN = 0
    LCD_Delay_ms(1);
}

/**
  * @brief  LCD1602写数据
  * @param  Data 要写入的数据
  * @retval 无
  */
static void LCD_WriteData(unsigned char Data)
{
    GPIO_SetBits(LCD_RS_GPIO, LCD_RS_PIN);    // RS = 1，写数据
    GPIO_ResetBits(LCD_RW_GPIO, LCD_RW_PIN);  // RW = 0，写操作
    
    // 写入数据
    LCD_DATA_GPIO->ODR = (LCD_DATA_GPIO->ODR & 0xFF00) | Data;
    
    GPIO_SetBits(LCD_EN_GPIO, LCD_EN_PIN);    // EN = 1
    LCD_Delay_ms(1);
    GPIO_ResetBits(LCD_EN_GPIO, LCD_EN_PIN);  // EN = 0
    LCD_Delay_ms(1);
}

/**
  * @brief  LCD1602设置光标位置
  * @param  Line 行位置，范围：1~2
  * @param  Column 列位置，范围：1~16
  * @retval 无
  */
static void LCD_SetCursor(unsigned char Line, unsigned char Column)
{
    if(Line == 1)
    {
        LCD_WriteCommand(0x80 | (Column - 1));
    }
    else if(Line == 2)
    {
        LCD_WriteCommand(0x80 | (Column - 1 + 0x40));
    }
}

/**
  * @brief  LCD1602 GPIO配置
  * @param  无
  * @retval 无
  */
static void LCD_GPIO_Config(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;
    
    // 使能时钟
    RCC_APB2PeriphClockCmd(LCD_RS_RCC | LCD_RW_RCC | LCD_EN_RCC | LCD_DATA_RCC, ENABLE);
    
    // 配置RS、RW、EN引脚
    GPIO_InitStructure.GPIO_Pin = LCD_RS_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(LCD_RS_GPIO, &GPIO_InitStructure);
    
    GPIO_InitStructure.GPIO_Pin = LCD_RW_PIN;
    GPIO_Init(LCD_RW_GPIO, &GPIO_InitStructure);
    
    GPIO_InitStructure.GPIO_Pin = LCD_EN_PIN;
    GPIO_Init(LCD_EN_GPIO, &GPIO_InitStructure);
    
    GPIO_InitStructure.GPIO_Pin = LCD_DATA_PINS;
    GPIO_Init(LCD_DATA_GPIO, &GPIO_InitStructure);
    
    GPIO_ResetBits(LCD_RS_GPIO, LCD_RS_PIN);
    GPIO_ResetBits(LCD_RW_GPIO, LCD_RW_PIN);
    GPIO_ResetBits(LCD_EN_GPIO, LCD_EN_PIN);
}

/**
  * @brief  LCD1602初始化函数
  * @param  无
  * @retval 无
  */
void LCD_Init(void)
{
    // GPIO初始化
    LCD_GPIO_Config();
    
    LCD_Delay_ms(50);  // 增加初始化延时
    
    LCD_WriteCommand(0x38); // 八位数据接口，两行显示，5*7点阵
    LCD_Delay_ms(5);
    
    LCD_WriteCommand(0x38); // 重复发送确保初始化成功
    LCD_Delay_ms(5);
    
    LCD_WriteCommand(0x38); // 第三次发送
    LCD_Delay_ms(5);
    
    LCD_WriteCommand(0x0c); // 显示开，光标关，闪烁关
    LCD_Delay_ms(5);
    
    LCD_WriteCommand(0x06); // 数据读写操作后，光标自动加一，画面不动
    LCD_Delay_ms(5);
    
    LCD_WriteCommand(0x01); // 光标复位，清屏
    LCD_Delay_ms(10); // 增加清屏后的延时
}

/**
  * @brief  LCD1602清屏函数
  * @param  无
  * @retval 无
  */
void LCD_Clear(void)
{
    LCD_WriteCommand(0x01); // 清屏命令
    LCD_Delay_ms(10);       // 清屏需要较长时间
}

/**
  * @brief  在LCD1602指定位置上显示一个字符
  * @param  Line 行位置，范围：1~2
  * @param  Column 列位置，范围：1~16
  * @param  Char 要显示的字符
  * @retval 无
  */
void LCD_ShowChar(unsigned char Line, unsigned char Column, char Char)
{
    LCD_SetCursor(Line, Column);
    LCD_WriteData(Char);
}

/**
  * @brief  在LCD1602指定位置开始显示所给字符串
  * @param  Line 起始行位置，范围：1~2
  * @param  Column 起始列位置，范围：1~16
  * @param  String 要显示的字符串
  * @retval 无
  */
void LCD_ShowString(unsigned char Line, unsigned char Column, char *String)
{
    unsigned char i;
    LCD_SetCursor(Line, Column);
    for(i = 0; String[i] != '\0'; i++)
    {
        LCD_WriteData(String[i]);
    }
}

/**
  * @brief  返回值=X的Y次方
  */
static int LCD_Pow(int X, int Y)
{
    unsigned char i;
    int Result = 1;
    for(i = 0; i < Y; i++)
    {
        Result *= X;
    }
    return Result;
}

/**
  * @brief  在LCD1602指定位置开始显示所给数字
  * @param  Line 起始行位置，范围：1~2
  * @param  Column 起始列位置，范围：1~16
  * @param  Number 要显示的数字，范围：0~65535
  * @param  Length 要显示数字的长度，范围：1~5
  * @retval 无
  */
void LCD_ShowNum(unsigned char Line, unsigned char Column, unsigned int Number, unsigned char Length)
{
    unsigned char i;
    LCD_SetCursor(Line, Column);
    for(i = Length; i > 0; i--)
    {
        LCD_WriteData(Number / LCD_Pow(10, i - 1) % 10 + '0');
    }
}

/**
  * @brief  在LCD1602指定位置开始以有符号十进制显示所给数字
  * @param  Line 起始行位置，范围：1~2
  * @param  Column 起始列位置，范围：1~16
  * @param  Number 要显示的数字，范围：-32768~32767
  * @param  Length 要显示数字的长度，范围：1~5
  * @retval 无
  */
void LCD_ShowSignedNum(unsigned char Line, unsigned char Column, int Number, unsigned char Length)
{
    unsigned char i;
    unsigned int Number1;
    LCD_SetCursor(Line, Column);
    if(Number >= 0)
    {
        LCD_WriteData('+');
        Number1 = Number;
    }
    else
    {
        LCD_WriteData('-');
        Number1 = -Number;
    }
    for(i = Length; i > 0; i--)
    {
        LCD_WriteData(Number1 / LCD_Pow(10, i - 1) % 10 + '0');
    }
}

/**
  * @brief  在LCD1602指定位置开始以十六进制显示所给数字
  * @param  Line 起始行位置，范围：1~2
  * @param  Column 起始列位置，范围：1~16
  * @param  Number 要显示的数字，范围：0~0xFFFF
  * @param  Length 要显示数字的长度，范围：1~4
  * @retval 无
  */
void LCD_ShowHexNum(unsigned char Line, unsigned char Column, unsigned int Number, unsigned char Length)
{
    unsigned char i, SingleNumber;
    LCD_SetCursor(Line, Column);
    for(i = Length; i > 0; i--)
    {
        SingleNumber = Number / LCD_Pow(16, i - 1) % 16;
        if(SingleNumber < 10)
        {
            LCD_WriteData(SingleNumber + '0');
        }
        else
        {
            LCD_WriteData(SingleNumber - 10 + 'A');
        }
    }
}

/**
  * @brief  在LCD1602指定位置开始以二进制显示所给数字
  * @param  Line 起始行位置，范围：1~2
  * @param  Column 起始列位置，范围：1~16
  * @param  Number 要显示的数字，范围：0~1111 1111 1111 1111
  * @param  Length 要显示数字的长度，范围：1~16
  * @retval 无
  */
void LCD_ShowBinNum(unsigned char Line, unsigned char Column, unsigned int Number, unsigned char Length)
{
    unsigned char i;
    LCD_SetCursor(Line, Column);
    for(i = Length; i > 0; i--)
    {
        LCD_WriteData(Number / LCD_Pow(2, i - 1) % 2 + '0');
    }
}
```

---
