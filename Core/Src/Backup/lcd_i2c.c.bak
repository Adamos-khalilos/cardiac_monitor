#include "lcd_i2c.h"

static I2C_HandleTypeDef *_hi2c;

// Bits du PCF8574
#define RS  0x01
#define RW  0x02
#define EN  0x04
#define BL  0x08  // Backlight

static void LCD_SendNibble(uint8_t nibble, uint8_t mode) {
    uint8_t data = (nibble & 0xF0) | mode | BL;
    uint8_t buf[4];
    buf[0] = data | EN;
    buf[1] = data | EN;
    buf[2] = data & ~EN;
    buf[3] = data & ~EN;
    HAL_I2C_Master_Transmit(_hi2c, LCD_ADDR, buf, 4, 10);
}

static void LCD_SendByte(uint8_t byte, uint8_t mode) {
    LCD_SendNibble(byte & 0xF0, mode);
    LCD_SendNibble((byte << 4) & 0xF0, mode);
    HAL_Delay(1);
}

void LCD_Init(I2C_HandleTypeDef *hi2c) {
    _hi2c = hi2c;
    HAL_Delay(50);

    // Séquence d'initialisation 4 bits
    LCD_SendNibble(0x30, 0); HAL_Delay(5);
    LCD_SendNibble(0x30, 0); HAL_Delay(1);
    LCD_SendNibble(0x30, 0); HAL_Delay(1);
    LCD_SendNibble(0x20, 0); HAL_Delay(1);  // Passe en 4 bits

    LCD_SendByte(0x28, 0);  // 4-bit, 2 lignes, 5x8
    LCD_SendByte(0x0C, 0);  // Display ON, curseur OFF
    LCD_SendByte(0x06, 0);  // Incrément auto
    LCD_SendByte(0x01, 0);  // Clear
    HAL_Delay(2);
}

void LCD_Clear(void) {
    LCD_SendByte(0x01, 0);
    HAL_Delay(2);
}

void LCD_SetCursor(uint8_t col, uint8_t row) {
    uint8_t addr = (row == 0) ? (0x80 + col) : (0xC0 + col);
    LCD_SendByte(addr, 0);
}

void LCD_Print(char *str) {
    while (*str) {
        LCD_SendByte((uint8_t)(*str++), RS);
    }
}
