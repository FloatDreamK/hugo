+++
date = '2025-12-05T20:52:54+08:00'
draft = false
title = '串口通信，AD转换，虚拟终端实时显示电位器的值，并且能控制LED灯的亮灭'
categories = ["单片机项目"]
+++

---

main.c
```c
#include "stm32f10x.h"
#include "stdio.h"
#include "led_control.h"
#include "uart.h"
#include "adc.h"

void Delay(__IO uint32_t nCount)
{
  while(nCount--);
}

int main(void)
{
  USART1_Init();
  LED_Init();
  ADC_Init_PA3();
  
  printf("hello world!\r\n");

  while(1) {
    if (USART1_CheckReceived()) {
      uint8_t received_char = USART1_ReceiveChar();
      LED_Toggle();
      printf("%c\r\n", received_char);
    }
    
    uint16_t voltage = ADC_GetVoltage();
    printf("PA3 V: %d.%03d V\r", voltage / 1000, voltage % 1000);
    Delay(0x7FFFF);
  }
}
```

adc.h
```c
#ifndef __ADC_H
#define __ADC_H

#include "stm32f10x.h"
#include "stdint.h"

#define ADC_CHANNEL   ADC_Channel_3

void ADC_Init_PA3(void);
uint16_t ADC_GetValue(void);
uint16_t ADC_GetVoltage(void);

#endif /* __ADC_H */
```

adc.c
```c
#include "adc.h"

void ADC_Init_PA3(void) {
    ADC_InitTypeDef ADC_InitStructure;
    GPIO_InitTypeDef GPIO_InitStructure;
    
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA | RCC_APB2Periph_ADC1, ENABLE);
    
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_3;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
    ADC_InitStructure.ADC_ScanConvMode = DISABLE;
    ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
    ADC_InitStructure.ADC_NbrOfChannel = 1;
    
    ADC_Init(ADC1, &ADC_InitStructure);
    
    ADC_RegularChannelConfig(ADC1, ADC_CHANNEL, 1, ADC_SampleTime_239Cycles5);
    
    ADC_Cmd(ADC1, ENABLE);
    
    ADC_ResetCalibration(ADC1);
    while(ADC_GetResetCalibrationStatus(ADC1));
    
    ADC_StartCalibration(ADC1);
    while(ADC_GetCalibrationStatus(ADC1));
    
    ADC_SoftwareStartConvCmd(ADC1, ENABLE);
}

uint16_t ADC_GetValue(void) {
    while(!ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC));
    
    return ADC_GetConversionValue(ADC1);
}

uint16_t ADC_GetVoltage(void) {
    uint16_t adc_value = ADC_GetValue();
    
    return (uint16_t)((adc_value * 3300) / 4095);
}
```

uart.h
```c
#ifndef __UART_H
#define __UART_H

#include "stm32f10x.h"
#include "stdint.h"

#define USART1_BAUDRATE    115200

void USART1_Init(void);
void USART1_SendChar(uint8_t ch);
void USART1_SendString(uint8_t *str);
uint8_t USART1_ReceiveChar(void);
uint8_t USART1_CheckReceived(void);

#endif /* __UART_H */
```

uart.c
```c
#include "uart.h"
#include "stdio.h"

void USART1_Init(void) {
    USART_InitTypeDef USART_InitStructure;
    GPIO_InitTypeDef GPIO_InitStructure;
    
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1 | RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
    GPIO_Init(GPIOA, &GPIO_InitStructure);
    
    USART_InitStructure.USART_BaudRate = USART1_BAUDRATE;
    USART_InitStructure.USART_WordLength = USART_WordLength_8b;
    USART_InitStructure.USART_StopBits = USART_StopBits_1;
    USART_InitStructure.USART_Parity = USART_Parity_No;
    USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
    USART_InitStructure.USART_Mode = USART_Mode_Rx | USART_Mode_Tx;
    
    USART_Init(USART1, &USART_InitStructure);
    USART_Cmd(USART1, ENABLE);
}

void USART1_SendChar(uint8_t ch) {
    while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
    USART_SendData(USART1, (uint16_t)ch);
}

void USART1_SendString(uint8_t *str) {
    while (*str) {
        USART1_SendChar(*str++);
    }
}

uint8_t USART1_ReceiveChar(void) {
    while (USART_GetFlagStatus(USART1, USART_FLAG_RXNE) == RESET);
    return (uint8_t)USART_ReceiveData(USART1);
}

uint8_t USART1_CheckReceived(void) {
    return (USART_GetFlagStatus(USART1, USART_FLAG_RXNE) != RESET) ? 1 : 0;
}

int fputc(int ch, FILE *f) {
    // if (ch == '\n') {
    //     USART1_SendChar('\r');
    // }
    USART1_SendChar((uint8_t)ch);
    return ch;
}
```

led_conrol.h
```c
#ifndef __LED_CONTROL_H
#define __LED_CONTROL_H

#include "stm32f10x.h"

#define LED_PORT    GPIOA
#define LED_PIN     GPIO_Pin_0

void LED_Init(void);
void LED_Toggle(void);

#endif /* __LED_CONTROL_H */
```

led_conrol.c
```c
#include "led_control.h"

void LED_Init(void) {
    GPIO_InitTypeDef GPIO_InitStructure;
    
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
    GPIO_InitStructure.GPIO_Pin = LED_PIN;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_Init(LED_PORT, &GPIO_InitStructure);
    
    GPIO_ResetBits(LED_PORT, LED_PIN);
}

void LED_Toggle(void) {
    if (GPIO_ReadOutputDataBit(LED_PORT, LED_PIN) == Bit_SET) {
        GPIO_ResetBits(LED_PORT, LED_PIN);
    } else {
        GPIO_SetBits(LED_PORT, LED_PIN);
    }
}
```

---
