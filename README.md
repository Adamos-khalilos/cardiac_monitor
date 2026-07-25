# Cardiac Monitor

An STM32F407-based cardiac monitoring system that samples a signal, classifies it in real time using NanoEdge AI, displays results on an LCD, logs data to an SD card (in both plain-text and AES-128-CBC encrypted form), and sends readings to an ESP32 over UART — which then publishes the data to the cloud via MQTT (HiveMQ) for the companion Android app to display.

## Features

- **Signal acquisition** — reads analog input via ADC1 (channel 1, PA1). Currently wired to a **potentiometer for testing/simulating a signal** (swap in a real ECG front-end for production use)
- **AI classification** — uses NanoEdge AI to classify samples into:
  - `NORMAL`
  - `BRADYCARDIE`
  - `TACHYCARDIE`
  - `ARYTHMIE`
- **LCD display (I2C)** — shows live ADC/voltage readings and classification results with confidence percentage
- **SD card logging (FATFS)**
  - `LOG_CSF.txt` — plain-text log of every reading and classification
  - `ENC_LOG.hex` — the same data encrypted with AES-128-CBC before being written, one self-contained `IV:ciphertext` line per entry
- **UART → ESP32 → MQTT (HiveMQ)** — the STM32 streams ADC values and classification results over UART (115200 baud) to an ESP32, which connects to WiFi and publishes the data to a HiveMQ MQTT broker
- **Android app** — companion app (in `android-app/`) that subscribes to the MQTT topic(s) on HiveMQ and displays live values from the device

## Hardware

- MCU: **STM32F407VGTx**
- Potentiometer on ADC1 channel 1 (PA1) — signal source for testing
- I2C LCD display
- SD card via SDIO
- UART2 link to an ESP32 (115200 baud)
- ESP32 — connects to WiFi, publishes sensor/classification data to HiveMQ over MQTT

## Data Flow

```
Potentiometer → STM32 ADC → NanoEdge AI classification
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   LCD display   SD card log   UART → ESP32
                (plain + AES)        │
                                     ▼
                              WiFi → MQTT publish
                                     │
                                     ▼
                              HiveMQ broker
                                     │
                                     ▼
                            Android app (MQTT subscriber)
```

## Project Structure

```
Core/                        Main application source (STM32CubeIDE)
Drivers/                     STM32 HAL and CMSIS drivers
FATFS/                       FAT filesystem middleware for SD card storage
Middlewares/                 Third-party middleware (FatFs source)
cardiac_monitor.ioc           STM32CubeMX configuration
cardiac_monitor.launch        Run configuration
cardiac_monitor Debug.launch  Debug configuration
STM32F407VGTX_FLASH.ld        Flash linker script
STM32F407VGTX_RAM.ld          RAM linker script
android-app/                  Companion Android app (APK) — MQTT subscriber for live monitoring
```

## How It Works

1. On boot, the LCD, SD card, and NanoEdge AI classifier are all initialized. Any failure shows an error message on the LCD and halts.
2. In the main loop, the device reads an ADC sample every 50ms, converts it to millivolts, and:
   - Displays the live value on the LCD
   - Logs it in plain text to `LOG_CSF.txt`
   - Encrypts and logs it to `ENC_LOG.hex`
   - Sends it over UART to the ESP32
3. Every 8 samples, the buffered window is passed to the NanoEdge AI classifier, which outputs a predicted class and confidence. This result is shown on the LCD, logged (plain + encrypted), and sent over UART to the ESP32.
4. The ESP32 receives this data over UART and publishes it to a HiveMQ MQTT broker over WiFi.
5. The Android app subscribes to the relevant MQTT topic(s) on HiveMQ and displays the live readings and classification to the user.

## Encryption Notes

Each log line in `ENC_LOG.hex` is independently encrypted (its own IV), formatted as:
```
<IV_hex>:<ciphertext_hex>
```
This means any single line can be decrypted on its own — you don't need the full file to recover one reading. The AES key is currently hardcoded in `main.c` (`aes_key`) — **change this before any real deployment**, and make sure the same key is used on the decrypting side (e.g. a Python script).

## Getting Started

1. Open `cardiac_monitor.ioc` in STM32CubeIDE / STM32CubeMX
2. Build and flash to an STM32F407VGTx board
3. Wire up the potentiometer, I2C LCD, SD card, and UART link to the ESP32
4. Flash the ESP32 with firmware that reads UART from the STM32 and publishes to your HiveMQ broker over MQTT
5. Install `android-app/app-debug.apk` on an Android device — configure it to subscribe to your HiveMQ broker/topic to view live data (you may need to enable "install from unknown sources")

## Author

Adamos-khalilos
