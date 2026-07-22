#ifndef LCD_I2C_H
#define LCD_I2C_H

#include "stm32f4xx_hal.h"
#include <string.h>
#include <stdio.h>

#define LCD_ADDR     (0x27 << 1)  // Change en 0x3F<<1 si nécessaire
#define LCD_COLS     16
#define LCD_ROWS     2

void LCD_Init(I2C_HandleTypeDef *hi2c);
void LCD_Clear(void);
void LCD_SetCursor(uint8_t col, uint8_t row);
void LCD_Print(char *str);

#endif
