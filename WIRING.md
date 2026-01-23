# Techack1 Complete Wiring Guide

> **⚠️ CRITICAL WARNING:** Incorrect wiring WILL permanently damage your PCB and components. Double-check ALL connections before applying power. Use a multimeter to verify there are no shorts between VCC and GND before powering on.

---

## Table of Contents
1. [Component List Reference](#component-list-reference)
2. [Power System](#power-system)
3. [ESP32-S3R8 Microcontroller (U8)](#esp32-s3r8-microcontroller-u8)
4. [W25Q64JVSSIQ Flash Memory (U9)](#w25q64jvssiq-flash-memory-u9)
5. [OLED Display - SSD1306 (H3)](#oled-display---ssd1306-h3)
6. [CC1101 Wireless Module 433MHz (H4)](#cc1101-wireless-module-433mhz-h4)
7. [USB Connections](#usb-connections)
8. [Micro SD Card Slot - TF-01A (CARD1)](#micro-sd-card-slot---tf-01a-card1)
9. [Buttons and Switches](#buttons-and-switches)
10. [LED Indicator (LED2)](#led-indicator-led2)
11. [Crystal Oscillator (X2)](#crystal-oscillator-x2)
12. [Complete Wiring Summary Tables](#complete-wiring-summary-tables)
13. [EasyEDA Net Names](#easyeda-net-names)
14. [Pre-Power Safety Checklist](#pre-power-safety-checklist)

---

## Component List Reference

This wiring guide is based on the following specific components from your parts list:

| Designator | Part Number | Description |
|------------|-------------|-------------|
| U8 | ESP32-S3R8 | Microcontroller (QFN56 package, 8MB PSRAM) |
| U9 | W25Q64JVSSIQ | 64Mbit (8MB) SPI Flash Memory (SOIC-8) |
| U11 | TP4056-42-ESOP8 | Li-Ion Battery Charger IC (ESOP-8) |
| U12 | AP2112K-3.3TRG1 | 3.3V 600mA LDO Regulator (SOT-25) |
| H3 | SSD1306 | 0.96" OLED Display 128x64 I2C (4-pin module) |
| H4 | CC1101 | 433MHz Wireless Module with SMA Antenna (2x4 header) |
| USB4 | TYPE-C 24P QCHT | USB Type-C Connector |
| USB3 | U-USBAR04P-M003 | USB-A Connector (for BadUSB) |
| CARD1 | TF-01A | Micro SD Card Slot |
| X2 | 40MHz Crystal | Crystal Oscillator |
| SW3/SW? | SPEH110100 | Tactile Push Button (Boot) |
| SW7 | SPEH110100 | Tactile Push Button (User) |
| SW5 | MST-12D18G4 | SPDT Slide Switch (Power) |
| LED2 | KT-0603YG | Green SMD LED (0603) |

---

## Power System

### Power Flow Overview

```
USB-C (5V VBUS)
      │
      ▼
┌─────────────┐
│   TP4056    │ ◄── Battery Charger
│    (U11)    │
└──────┬──────┘
       │ BAT
       ▼
┌─────────────┐
│  Li-Po      │ ◄── 3.7V Battery (via JST connector)
│  Battery    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    SW5      │ ◄── Power On/Off Slide Switch
│ MST-12D18G4 │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  AP2112K    │ ◄── 3.3V LDO Regulator
│    (U12)    │
└──────┬──────┘
       │ 3.3V
       ▼
   ┌───┴───┐
   │  ALL  │ ◄── ESP32, Flash, OLED, CC1101, SD Card
   │ 3.3V  │
   │ ICs   │
   └───────┘
```

---

### TP4056-42-ESOP8 (U11) - Battery Charger

The TP4056 is an 8-pin lithium-ion battery charger IC. The ESOP-8 package has the following pinout:

```
        TP4056-42-ESOP8 (U11)
        ┌────────────────────┐
        │                    │
  TEMP ─┤ 1              8 ├─ CE
        │                    │
  PROG ─┤ 2              7 ├─ CHRG
        │       EP          │
   GND ─┤ 3    (GND)     6 ├─ STDBY
        │                    │
   VCC ─┤ 4              5 ├─ BAT
        │                    │
        └────────────────────┘
          (Exposed Pad = GND)
```

#### TP4056 Wiring Table

| Pin | Name | Connect To | Description |
|:---:|:----:|------------|-------------|
| 1 | TEMP | 10kΩ resistor to GND | Temperature monitoring. Connect 10kΩ NTC thermistor, or 10kΩ fixed resistor to GND to disable temp monitoring |
| 2 | PROG | 1.2kΩ resistor to GND | Programs charge current. R = 1200/I(mA). 1.2kΩ = 1000mA charge current |
| 3 | GND | System GND | Ground |
| 4 | VCC | USB-C VBUS (5V) | Input power from USB (4.0V to 8.0V) |
| 5 | BAT | Battery+ / SW5 input | Connect to Li-Po positive and power switch |
| 6 | STDBY | 1kΩ → LED (optional) | Charge complete indicator, open-drain active LOW |
| 7 | CHRG | 1kΩ → LED (optional) | Charging indicator, open-drain active LOW |
| 8 | CE | VCC (tied HIGH) | Chip Enable, HIGH = enabled |
| EP | Exposed Pad | GND | Thermal pad, connect to ground plane |

#### TP4056 Required Components

| Component | Value | Connection | Purpose |
|-----------|-------|------------|---------|
| R_PROG | 1.2kΩ | Pin 2 to GND | Sets charge current to ~1A |
| R_TEMP | 10kΩ | Pin 1 to GND | Disables temp sensing (or use NTC) |
| C_IN | 10µF ceramic | VCC (Pin 4) to GND | Input bypass capacitor |
| C_BAT | 10µF ceramic | BAT (Pin 5) to GND | Output bypass capacitor |

---

### AP2112K-3.3TRG1 (U12) - 3.3V LDO Regulator

The AP2112K is a 600mA low dropout regulator in SOT-25 (5-pin) package:

```
        AP2112K-3.3TRG1 (U12)
        ┌─────────────────┐
        │       ┌───┐     │
   VIN ─┤ 1     │   │  5 ├─ VOUT
        │       └───┘     │
   GND ─┤ 2            4 ├─ NC
        │                 │
    EN ─┤ 3               │
        │                 │
        └─────────────────┘
```

#### AP2112K Wiring Table

| Pin | Name | Connect To | Description |
|:---:|:----:|------------|-------------|
| 1 | VIN | SW5 output (Battery voltage) | Input voltage (2.5V to 6.0V) |
| 2 | GND | System GND | Ground |
| 3 | EN | VIN (tied to input) | Enable pin, HIGH = enabled |
| 4 | NC | Not Connected | Leave floating |
| 5 | VOUT | VCC_3V3 net (to all ICs) | 3.3V regulated output |

#### AP2112K Required Components

| Component | Value | Connection | Purpose |
|-----------|-------|------------|---------|
| C_IN | 10µF ceramic | VIN (Pin 1) to GND | Input capacitor (required) |
| C_OUT | 10µF ceramic | VOUT (Pin 5) to GND | Output capacitor (required) |

> **Note:** Place capacitors as close to the IC as possible.

---

### SW5 - MST-12D18G4 Power Switch

SPDT slide switch for power control:

```
    MST-12D18G4 (SW5)
    ┌────────────────────┐
    │  ═══════════════   │  ← Slider
    │                    │
    │   1     2     3    │
    │   ●     ●     ●    │
    └───┬─────┬─────┬────┘
        │     │     │
       NO   COM    NC
```

| Pin | Name | Connect To | Description |
|:---:|:----:|------------|-------------|
| 1 | NO (Normally Open) | AP2112K VIN (Pin 1) | Output to LDO |
| 2 | COM (Common) | TP4056 BAT (Pin 5) | Battery voltage input |
| 3 | NC (Normally Closed) | Not Connected | Leave floating |

**Operation:** Slide switch to connect COM to NO = Power ON

---

## ESP32-S3R8 Microcontroller (U8)

The ESP32-S3R8 is a QFN56 package with embedded 8MB Octal PSRAM. This is the **bare chip**, not a module.

### ESP32-S3R8 Key Pin Groups

Based on the ESP32-S3 datasheet (QFN56 package):

#### Power Supply Pins

| Pin | Name | Connect To | Description |
|:---:|:----:|------------|-------------|
| 2, 3 | VDD3P3 | VCC_3V3 | Main 3.3V power supply |
| 20 | VDD3P3_RTC | VCC_3V3 via 100nF | RTC domain power |
| 29 | VDD_SPI | VCC_3V3 | SPI Flash voltage domain |
| 46 | VDD3P3_CPU | VCC_3V3 via 100nF | CPU power domain |
| 55, 56 | VDDA | VCC_3V3 via ferrite bead + 100nF | Analog power |
| 57 | GND | GND | Ground (also pad on bottom) |

#### USB OTG Pins (Native USB for programming and BadUSB)

| Pin | Name | GPIO | Connect To | Description |
|:---:|:----:|:----:|------------|-------------|
| 25 | USB_D- | GPIO19 | USB-C D- & USB-A D- | USB Data Minus |
| 26 | USB_D+ | GPIO20 | USB-C D+ & USB-A D+ | USB Data Plus |

#### Crystal Oscillator Pins

| Pin | Name | Connect To | Description |
|:---:|:----:|------------|-------------|
| 53 | XTAL_N | X2 Pin 2 + 10pF to GND | Crystal output |
| 54 | XTAL_P | X2 Pin 1 + 10pF to GND | Crystal input |

#### SPI Flash Pins (for W25Q64JVSSIQ - U9)

These pins are dedicated for external SPI flash on ESP32-S3R8:

| Pin | Name | GPIO | Connect To | Description |
|:---:|:----:|:----:|------------|-------------|
| 28 | SPICS1 | GPIO26 | Not used (CS1) | Secondary flash CS |
| 30 | SPIHD | GPIO27 | U9 HOLD# (Pin 7) | SPI Hold |
| 31 | SPIWP | GPIO28 | U9 WP# (Pin 3) | SPI Write Protect |
| 32 | SPICS0 | GPIO29 | U9 CS# (Pin 1) | Chip Select |
| 33 | SPICLK | GPIO30 | U9 CLK (Pin 6) | SPI Clock |
| 34 | SPIQ | GPIO31 | U9 DO (Pin 2) | SPI MISO |
| 35 | SPID | GPIO32 | U9 DI (Pin 5) | SPI MOSI |

#### HSPI (SPI2) for CC1101 and SD Card

These GPIO pins can be configured for a second SPI bus:

| Pin | Name | GPIO | Function | Connect To |
|:---:|:----:|:----:|:--------:|------------|
| 17 | GPIO12 | GPIO12 | FSPICLK (SPI2_CLK) | CC1101 SCK, SD CLK |
| 18 | GPIO13 | GPIO13 | FSPIQ (SPI2_MISO) | CC1101 MISO, SD DAT0 |
| 16 | GPIO11 | GPIO11 | FSPID (SPI2_MOSI) | CC1101 MOSI, SD CMD |
| 15 | GPIO10 | GPIO10 | FSPICS0 (SPI2_CS0) | CC1101 CSN |
| 19 | GPIO14 | GPIO14 | GPIO (CS1) | SD Card CS |

#### I2C Pins (for OLED Display)

| Pin | Name | GPIO | Connect To | Description |
|:---:|:----:|:----:|------------|-------------|
| 13 | GPIO8 | GPIO8 | H3 SDA (Pin 3) | I2C Data |
| 14 | GPIO9 | GPIO9 | H3 SCL (Pin 4) | I2C Clock |

#### CC1101 Control Pins

| Pin | Name | GPIO | Connect To | Description |
|:---:|:----:|:----:|------------|-------------|
| 7 | GPIO2 | GPIO2 | H4 GDO0 (Pin 3) | CC1101 Interrupt 0 |
| 8 | GPIO3 | GPIO3 | H4 GDO2 (Pin 2) | CC1101 Interrupt 2 |

#### Button and LED Pins

| Pin | Name | GPIO | Connect To | Description |
|:---:|:----:|:----:|------------|-------------|
| 5 | GPIO0 | GPIO0 | SW3 (Boot button) | Boot mode select (pull-up 10kΩ) |
| 9 | GPIO4 | GPIO4 | SW7 (User button) | User input (pull-up 10kΩ) |
| 52 | GPIO46 | GPIO46 | LED2 via resistor | LED control |

#### Strapping Pins (Critical for Boot Mode)

| GPIO | Function | Required State | How to Achieve |
|:----:|:--------:|:--------------:|----------------|
| GPIO0 | Boot Mode | HIGH (normal boot) | 10kΩ pull-up to 3V3 |
| GPIO45 | VDD_SPI Voltage | LOW (3.3V SPI) | 10kΩ pull-down to GND |
| GPIO46 | Boot Mode | LOW | 10kΩ pull-down to GND |

> **⚠️ Critical:** GPIO45 and GPIO46 MUST be pulled LOW or floating for normal operation.

---

## W25Q64JVSSIQ Flash Memory (U9)

8MB (64Mbit) SPI NOR Flash in SOIC-8 package:

```
        W25Q64JVSSIQ (U9)
        ┌──────────────────┐
        │                  │
   /CS ─┤ 1            8 ├─ VCC (3.3V)
        │                  │
    DO ─┤ 2            7 ├─ /HOLD
        │                  │
   /WP ─┤ 3            6 ├─ CLK
        │                  │
   GND ─┤ 4            5 ├─ DI
        │                  │
        └──────────────────┘
```

#### W25Q64JVSSIQ Wiring Table

| Pin | Name | Connect To | Description |
|:---:|:----:|------------|-------------|
| 1 | /CS | ESP32 GPIO29 (SPICS0) | Chip Select (active LOW) |
| 2 | DO (MISO) | ESP32 GPIO31 (SPIQ) | Data Output |
| 3 | /WP | 10kΩ pull-up to VCC | Write Protect (active LOW, pull HIGH to disable) |
| 4 | GND | System GND | Ground |
| 5 | DI (MOSI) | ESP32 GPIO32 (SPID) | Data Input |
| 6 | CLK | ESP32 GPIO30 (SPICLK) | SPI Clock |
| 7 | /HOLD | 10kΩ pull-up to VCC | Hold (active LOW, pull HIGH to disable) |
| 8 | VCC | VCC_3V3 | Power supply (2.7V to 3.6V) |

#### W25Q64JVSSIQ Required Components

| Component | Value | Connection | Purpose |
|-----------|-------|------------|---------|
| C_FLASH | 100nF ceramic | VCC (Pin 8) to GND (Pin 4) | Decoupling capacitor |
| R_WP | 10kΩ | /WP (Pin 3) to VCC | Pull-up resistor |
| R_HOLD | 10kΩ | /HOLD (Pin 7) to VCC | Pull-up resistor |

---

## OLED Display - SSD1306 (H3)

0.96" 128x64 I2C OLED Display Module (4-pin interface):

```
    SSD1306 OLED Module (H3)
    ┌─────────────────────────┐
    │  ┌─────────────────┐    │
    │  │                 │    │
    │  │   128x64 OLED   │    │
    │  │                 │    │
    │  └─────────────────┘    │
    │                         │
    │   GND  VCC  SCL  SDA    │
    │    1    2    3    4     │
    └────┬────┬────┬────┬─────┘
         │    │    │    │
```

#### SSD1306 OLED Wiring Table

| Pin | Name | Connect To | Description |
|:---:|:----:|------------|-------------|
| 1 | GND | System GND | Ground |
| 2 | VCC | VCC_3V3 | Power (3.3V compatible) |
| 3 | SCL | ESP32 GPIO9 | I2C Clock |
| 4 | SDA | ESP32 GPIO8 | I2C Data |

#### I2C Pull-up Resistors

| Component | Value | Connection | Purpose |
|-----------|-------|------------|---------|
| R_SDA | 4.7kΩ | SDA line to VCC_3V3 | I2C pull-up |
| R_SCL | 4.7kΩ | SCL line to VCC_3V3 | I2C pull-up |

> **Note:** Many SSD1306 modules include on-board pull-ups. Verify your module before adding external ones.

**I2C Address:** 0x3C (default) or 0x3D (depends on module configuration)

---

## CC1101 Wireless Module 433MHz (H4)

CC1101 sub-GHz transceiver module with 2x4 pin header (2.0mm pitch):

```
       CC1101 Module (H4) - 2.0-2*4P Header
       ┌─────────────────────────────────┐
       │                                 │
       │  ┌───────────────────────────┐  │
       │  │      CC1101 IC            │  │
       │  └───────────────────────────┘  │
       │                                 │
       │  ╔═════╗    ┌──────────────┐   │
       │  ║ SMA ║────│ 433MHz       │   │
       │  ╚═════╝    │ Antenna      │   │
       │             └──────────────┘   │
       │                                 │
       │  Top View of Header:           │
       │  ┌────┬────┬────┬────┐        │
       │  │ 1  │ 2  │ 3  │ 4  │        │
       │  │VCC │GDO2│GDO0│CSN │        │
       │  ├────┼────┼────┼────┤        │
       │  │ 5  │ 6  │ 7  │ 8  │        │
       │  │SCK │MOSI│MISO│GND │        │
       │  └────┴────┴────┴────┘        │
       │                                 │
       └─────────────────────────────────┘
```

> **⚠️ WARNING:** Verify your specific CC1101 module pinout! Some modules have different pin arrangements. Always check the silkscreen or datasheet for YOUR module.

#### CC1101 Wiring Table

| Header Pin | Name | Connect To | Description |
|:----------:|:----:|------------|-------------|
| 1 | VCC | VCC_3V3 | **3.3V ONLY!** (1.8V to 3.6V) |
| 2 | GDO2 | ESP32 GPIO3 | General Digital Output 2 (interrupt) |
| 3 | GDO0 | ESP32 GPIO2 | General Digital Output 0 (interrupt) |
| 4 | CSN | ESP32 GPIO10 (SPI2_CS0) | Chip Select (active LOW) |
| 5 | SCK | ESP32 GPIO12 (SPI2_CLK) | SPI Clock |
| 6 | MOSI | ESP32 GPIO11 (SPI2_MOSI) | SPI Data In (to CC1101) |
| 7 | MISO | ESP32 GPIO13 (SPI2_MISO) | SPI Data Out (from CC1101) |
| 8 | GND | System GND | Ground |

> **⚠️ CRITICAL:** The CC1101 operates at **3.3V ONLY**. Applying 5V will permanently destroy the module!

#### CC1101 Required Components

| Component | Value | Connection | Purpose |
|-----------|-------|------------|---------|
| C_CC1101 | 100nF ceramic | VCC (Pin 1) to GND (Pin 8) | Decoupling capacitor |

**Important Notes:**
- Always connect a proper 433MHz antenna before transmitting
- GDO0 and GDO2 are configurable - they can be set for TX/RX status, packet status, etc.

---

## USB Connections

### USB4 - TYPE-C 24P QCHT USB Connector

USB Type-C connector for charging and programming:

#### Essential USB-C Connections

| USB-C Pin | Name | Connect To | Description |
|:---------:|:----:|------------|-------------|
| A1, B12 | GND | System GND | Ground |
| A4, B9 | VBUS | TP4056 VCC (Pin 4) | 5V power input |
| A5 | CC1 | 5.1kΩ to GND | Configuration Channel 1 |
| B5 | CC2 | 5.1kΩ to GND | Configuration Channel 2 |
| A6, B6 | D+ | ESP32 GPIO20 (USB_D+) | USB Data Plus |
| A7, B7 | D- | ESP32 GPIO19 (USB_D-) | USB Data Minus |
| A12, B1 | GND | System GND | Ground |
| Shell | Shield | GND | EMI shielding |

> **⚠️ Critical:** The 5.1kΩ resistors on CC1 and CC2 are **mandatory** for USB-C to provide power. Without them, no power will be supplied.

#### USB-C Required Components

| Component | Value | Connection | Purpose |
|-----------|-------|------------|---------|
| R_CC1 | 5.1kΩ | CC1 (A5) to GND | USB-C sink identification |
| R_CC2 | 5.1kΩ | CC2 (B5) to GND | USB-C sink identification |

---

### USB3 - U-USBAR04P-M003 USB-A Connector (BadUSB)

USB-A Female connector for BadUSB functionality (device appears as keyboard/mouse):

```
    USB-A Female Connector (USB3)
    ┌─────────────────────────────┐
    │  ╔═══════════════════════╗  │
    │  ║                       ║  │
    │  ║       USB-A Female    ║  │
    │  ║                       ║  │
    │  ╚═══════════════════════╝  │
    │                             │
    │      1    2    3    4       │
    │     VBUS  D-   D+  GND      │
    └──────┬────┬────┬────┬───────┘
           │    │    │    │
```

#### USB-A Wiring Table

| Pin | Name | Connect To | Description |
|:---:|:----:|------------|-------------|
| 1 | VBUS | 5V supply (optional) | Can provide 5V to target device |
| 2 | D- | ESP32 GPIO19 (USB_D-) | USB Data Minus |
| 3 | D+ | ESP32 GPIO20 (USB_D+) | USB Data Plus |
| 4 | GND | System GND | Ground |
| Shell | Shield | GND | EMI shielding |

> **Note:** USB-C and USB-A share the same D+/D- lines from the ESP32-S3. Only one USB connection should be active at a time. Consider adding a MOSFET switch to control USB-A VBUS output.

---

## Micro SD Card Slot - TF-01A (CARD1)

Micro SD card slot using SPI mode:

```
       TF-01A Micro SD Slot (CARD1)
       ┌─────────────────────────────────┐
       │   ┌─────────────────────────┐   │
       │   │                         │   │
       │   │     Micro SD Card       │◄──│── Card Insert
       │   │                         │   │
       │   └─────────────────────────┘   │
       │                                 │
       │   Pin: 1  2  3  4  5  6  7  8   │
       └─────────────────────────────────┘
```

#### TF-01A SD Card Wiring Table (SPI Mode)

| Pin | Name | Connect To | Description |
|:---:|:----:|------------|-------------|
| 1 | DAT2 | 10kΩ pull-up to VCC | Not used in SPI mode |
| 2 | CD/DAT3 | ESP32 GPIO14 | Chip Select (active LOW) |
| 3 | CMD | ESP32 GPIO11 (SPI2_MOSI) | SPI MOSI |
| 4 | VDD | VCC_3V3 | Power supply (3.3V) |
| 5 | CLK | ESP32 GPIO12 (SPI2_CLK) | SPI Clock |
| 6 | VSS | System GND | Ground |
| 7 | DAT0 | ESP32 GPIO13 (SPI2_MISO) | SPI MISO |
| 8 | DAT1 | 10kΩ pull-up to VCC | Not used in SPI mode |

#### SD Card Required Components

| Component | Value | Connection | Purpose |
|-----------|-------|------------|---------|
| C_SD | 10µF ceramic | VDD (Pin 4) to VSS (Pin 6) | Power bypass |
| C_SD2 | 100nF ceramic | VDD (Pin 4) to VSS (Pin 6) | High-freq decoupling |
| R_DAT1 | 10kΩ | DAT1 (Pin 8) to VCC | Pull-up |
| R_DAT2 | 10kΩ | DAT2 (Pin 1) to VCC | Pull-up |

> **Note:** The SD card shares SPI2 bus with CC1101. Each device has its own CS (Chip Select) line.

---

## Buttons and Switches

### SW3/SW? - SPEH110100 Boot Button

Tactile push button for entering boot/flash mode:

```
       SPEH110100 Tactile Button (SW3)
       ┌───────────────┐
       │    ┌─────┐    │
       │    │     │    │
       │    └─────┘    │
       │               │
       │  1         2  │
       │  ●         ●  │
       │               │
       │  3         4  │
       │  ●         ●  │
       └───────────────┘
       
       Pins 1-2 internally connected
       Pins 3-4 internally connected
       Press connects 1-3 and 2-4
```

#### Boot Button (SW3) Wiring

| Connection | From | To | Description |
|:----------:|:----:|:--:|-------------|
| Button | Pin 1 or 2 | ESP32 GPIO0 | Button input |
| Button | Pin 3 or 4 | GND | Ground when pressed |
| Pull-up | GPIO0 | 10kΩ to VCC_3V3 | Keep HIGH during normal boot |

**Function:** Hold during power-on or reset to enter USB download mode

---

### SW7 - SPEH110100 User Button

Tactile push button for user input:

#### User Button (SW7) Wiring

| Connection | From | To | Description |
|:----------:|:----:|:--:|-------------|
| Button | Pin 1 or 2 | ESP32 GPIO4 | Button input |
| Button | Pin 3 or 4 | GND | Ground when pressed |
| Pull-up | GPIO4 | 10kΩ to VCC_3V3 | Keep HIGH when not pressed |

---

## LED Indicator (LED2)

### KT-0603YG Green SMD LED

0603 package green LED for status indication:

```
     Current Limiting Circuit
     
     ESP32 GPIO46 ───┬─── R_LED ───┬─── LED2 (Anode +)
                     │   (150Ω)    │
                     │             └─── LED2 (Cathode -) ─── GND
                     │
                     └─── [Active HIGH = LED ON]
```

**OR** (Alternative - Active LOW):

```
     VCC_3V3 ─── R_LED ───┬─── LED2 (Anode +)
               (150Ω)     │
                          └─── LED2 (Cathode -) ─── ESP32 GPIO46
                          
                          [Active LOW = LED ON]
```

#### LED Resistor Calculation

| Parameter | Value |
|-----------|-------|
| Supply Voltage | 3.3V |
| LED Forward Voltage (Green) | ~2.0V |
| Desired Current | 5-10mA |
| Resistor Value | R = (3.3V - 2.0V) / 10mA = 130Ω → **Use 150Ω standard value** |

---

## Crystal Oscillator (X2)

### 40MHz Crystal for ESP32-S3R8

```
     40MHz Crystal Circuit
     
     ESP32 XTAL_P ────┬──── C1 (10pF) ──── GND
        (Pin 54)      │
                  ┌───┴───┐
                  │ XTAL  │
                  │ 40MHz │
                  └───┬───┘
                      │
     ESP32 XTAL_N ────┴──── C2 (10pF) ──── GND
        (Pin 53)
```

#### Crystal Wiring Table

| Crystal Pin | Connect To | Description |
|:-----------:|------------|-------------|
| Pin 1 | ESP32 XTAL_P (Pin 54) | Crystal input |
| Pin 2 | ESP32 XTAL_N (Pin 53) | Crystal output |

#### Crystal Required Components

| Component | Value | Connection | Purpose |
|-----------|-------|------------|---------|
| X2 | 40MHz | Pins 53 & 54 | Main clock source |
| C_XTAL1 | 10pF | XTAL_P to GND | Load capacitor |
| C_XTAL2 | 10pF | XTAL_N to GND | Load capacitor |

> **Note:** Keep crystal traces short and symmetric. Place close to ESP32-S3R8.

---

## Complete Wiring Summary Tables

### Power Connections

| From | To | Net Name | Description |
|:----:|:--:|:--------:|-------------|
| USB-C VBUS | TP4056 Pin 4 (VCC) | VBUS_5V | USB 5V input |
| TP4056 Pin 5 (BAT) | Battery+ | BAT+ | Battery charge output |
| TP4056 Pin 5 (BAT) | SW5 Pin 2 (COM) | BAT+ | Battery to switch |
| SW5 Pin 1 (NO) | AP2112K Pin 1 (VIN) | VIN_SW | Switched battery |
| AP2112K Pin 5 (VOUT) | All 3.3V loads | VCC_3V3 | Regulated 3.3V |
| All GND pins | Common ground plane | GND | Ground |

### SPI Bus Connections

| Signal | ESP32 GPIO | W25Q64 (U9) | CC1101 (H4) | SD Card (CARD1) |
|:------:|:----------:|:-----------:|:-----------:|:---------------:|
| Flash CS | GPIO29 | Pin 1 (CS) | - | - |
| Flash CLK | GPIO30 | Pin 6 (CLK) | - | - |
| Flash MISO | GPIO31 | Pin 2 (DO) | - | - |
| Flash MOSI | GPIO32 | Pin 5 (DI) | - | - |
| SPI2 CS0 | GPIO10 | - | Pin 4 (CSN) | - |
| SPI2 CS1 | GPIO14 | - | - | Pin 2 (CS) |
| SPI2 CLK | GPIO12 | - | Pin 5 (SCK) | Pin 5 (CLK) |
| SPI2 MISO | GPIO13 | - | Pin 7 (MISO) | Pin 7 (DAT0) |
| SPI2 MOSI | GPIO11 | - | Pin 6 (MOSI) | Pin 3 (CMD) |

### I2C Bus Connections

| Signal | ESP32 GPIO | OLED (H3) | Pull-up |
|:------:|:----------:|:---------:|:-------:|
| SDA | GPIO8 | Pin 4 | 4.7kΩ to VCC |
| SCL | GPIO9 | Pin 3 | 4.7kΩ to VCC |

### GPIO Summary

| ESP32 GPIO | Function | Connected To |
|:----------:|:--------:|:------------:|
| GPIO0 | Boot Button | SW3 (with 10kΩ pull-up) |
| GPIO2 | CC1101 GDO0 | H4 Pin 3 |
| GPIO3 | CC1101 GDO2 | H4 Pin 2 |
| GPIO4 | User Button | SW7 (with 10kΩ pull-up) |
| GPIO8 | I2C SDA | H3 Pin 4 |
| GPIO9 | I2C SCL | H3 Pin 3 |
| GPIO10 | SPI2 CS (CC1101) | H4 Pin 4 |
| GPIO11 | SPI2 MOSI | H4 Pin 6, CARD1 Pin 3 |
| GPIO12 | SPI2 CLK | H4 Pin 5, CARD1 Pin 5 |
| GPIO13 | SPI2 MISO | H4 Pin 7, CARD1 Pin 7 |
| GPIO14 | SD Card CS | CARD1 Pin 2 |
| GPIO19 | USB D- | USB-C D-, USB-A D- |
| GPIO20 | USB D+ | USB-C D+, USB-A D+ |
| GPIO29 | Flash CS | U9 Pin 1 |
| GPIO30 | Flash CLK | U9 Pin 6 |
| GPIO31 | Flash MISO | U9 Pin 2 |
| GPIO32 | Flash MOSI | U9 Pin 5 |
| GPIO46 | LED | LED2 (via 150Ω) |

---

## EasyEDA Net Names

Use these net labels in EasyEDA for clean schematic organization:

| Net Name | Description | Color |
|:--------:|-------------|:-----:|
| `VBUS_5V` | USB 5V input power | Orange |
| `BAT+` | Battery positive terminal | Red |
| `VIN_SW` | Switched battery voltage | Dark Red |
| `VCC_3V3` | 3.3V regulated power | Red |
| `GND` | Ground | Black |
| `SPI2_CLK` | SPI2 Clock line | Yellow |
| `SPI2_MOSI` | SPI2 Master Out | Blue |
| `SPI2_MISO` | SPI2 Master In | Green |
| `SPI2_CS_CC1101` | CC1101 Chip Select | Purple |
| `SPI2_CS_SD` | SD Card Chip Select | Purple |
| `I2C_SDA` | I2C Data | Cyan |
| `I2C_SCL` | I2C Clock | Cyan |
| `USB_DP` | USB Data Plus | Blue |
| `USB_DM` | USB Data Minus | Blue |
| `CC1101_GDO0` | CC1101 Interrupt 0 | Orange |
| `CC1101_GDO2` | CC1101 Interrupt 2 | Orange |

---

## Pre-Power Safety Checklist

### Before Soldering

- [ ] Verify all component footprints match your actual parts
- [ ] Double-check component orientation (IC pin 1, LED polarity, etc.)
- [ ] Verify ESP32-S3R8 is the QFN56 bare chip (not a module)
- [ ] Confirm CC1101 module pinout matches your specific module

### Before First Power-On

- [ ] **Check for shorts:** Use multimeter continuity between VCC_3V3 and GND - should be OPEN
- [ ] **Check for shorts:** Use multimeter continuity between VBUS_5V and GND - should be OPEN
- [ ] **Verify USB-C CC resistors:** 5.1kΩ on CC1 and CC2
- [ ] **Verify strapping pins:** GPIO45 and GPIO46 pulled LOW or floating
- [ ] **Verify pull-ups:** GPIO0 has 10kΩ to 3V3
- [ ] **Verify decoupling caps:** All ICs have bypass capacitors
- [ ] **No battery connected yet**

### First Power-On Procedure

1. **Do NOT insert battery**
2. Connect USB-C to a current-limited power supply (set 100mA limit initially)
3. Check current draw (should be less than 50mA without battery)
4. Measure voltage on VCC_3V3 net (should be 3.3V ±0.1V)
5. Check for excessive heat on any component
6. If all good, increase current limit to 500mA
7. Disconnect USB-C
8. Connect battery (verify polarity!)
9. Turn on power switch (SW5)
10. Verify 3.3V on VCC_3V3 net again

### Thermal Check (After 1 Minute Running)

- [ ] TP4056: Warm (hot only when actively charging)
- [ ] AP2112K: Cool to slightly warm
- [ ] ESP32-S3R8: Warm
- [ ] CC1101: Cool to slightly warm
- [ ] **No component should be too hot to touch**

---

## Quick Reference Card

```
╔═════════════════════════════════════════════════════════════════════════════╗
║                    TECHACK1 QUICK WIRING REFERENCE                          ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  POWER CHAIN                        │  ESP32-S3R8 KEY PINS                  ║
║  ────────────                       │  ─────────────────                    ║
║  USB-C VBUS ──► TP4056 VCC (Pin 4)  │  GPIO0  = Boot Button (10k pull-up)   ║
║  TP4056 BAT ──► SW5 COM             │  GPIO2  = CC1101 GDO0                 ║
║  SW5 NO     ──► AP2112K VIN         │  GPIO3  = CC1101 GDO2                 ║
║  AP2112K OUT ─► VCC_3V3 (all ICs)   │  GPIO4  = User Button (10k pull-up)   ║
║                                     │  GPIO8  = I2C SDA (OLED)              ║
║  CC1101 MODULE (H4)                 │  GPIO9  = I2C SCL (OLED)              ║
║  ───────────────                    │  GPIO10 = CC1101 CS                   ║
║  Pin 1 VCC  ──► VCC_3V3 (3.3V!)     │  GPIO11 = SPI2 MOSI                   ║
║  Pin 2 GDO2 ──► GPIO3               │  GPIO12 = SPI2 CLK                    ║
║  Pin 3 GDO0 ──► GPIO2               │  GPIO13 = SPI2 MISO                   ║
║  Pin 4 CSN  ──► GPIO10              │  GPIO14 = SD Card CS                  ║
║  Pin 5 SCK  ──► GPIO12              │  GPIO19 = USB D-                      ║
║  Pin 6 MOSI ──► GPIO11              │  GPIO20 = USB D+                      ║
║  Pin 7 MISO ──► GPIO13              │  GPIO29 = Flash CS                    ║
║  Pin 8 GND  ──► GND                 │  GPIO46 = LED (via 150Ω)              ║
║                                     │                                       ║
║  OLED DISPLAY (H3)                  │  W25Q64 FLASH (U9)                    ║
║  ─────────────────                  │  ────────────────                     ║
║  Pin 1 GND ──► GND                  │  Pin 1 /CS   ──► GPIO29               ║
║  Pin 2 VCC ──► VCC_3V3              │  Pin 2 DO    ──► GPIO31               ║
║  Pin 3 SCL ──► GPIO9                │  Pin 3 /WP   ──► 10kΩ to VCC          ║
║  Pin 4 SDA ──► GPIO8                │  Pin 5 DI    ──► GPIO32               ║
║                                     │  Pin 6 CLK   ──► GPIO30               ║
║  SD CARD (CARD1)                    │  Pin 7 /HOLD ──► 10kΩ to VCC          ║
║  ───────────────                    │  Pin 8 VCC   ──► VCC_3V3              ║
║  Pin 2 CS  ──► GPIO14               │                                       ║
║  Pin 3 CMD ──► GPIO11               │  ⚠️  CRITICAL WARNINGS:               ║
║  Pin 4 VDD ──► VCC_3V3              │  • CC1101 is 3.3V ONLY - NO 5V!       ║
║  Pin 5 CLK ──► GPIO12               │  • USB-C needs 5.1kΩ on CC1/CC2       ║
║  Pin 7 DAT0 ► GPIO13                │  • Check for shorts before power!     ║
║                                     │  • GPIO45/46 must be LOW at boot      ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

---

**Document Version:** 1.0  
**Last Updated:** January 2025  
**Based On:** Official datasheets for ESP32-S3R8, W25Q64JVSSIQ, TP4056, AP2112K, CC1101, SSD1306

> **Disclaimer:** This wiring guide is based on datasheet specifications. Always verify connections against your specific component datasheets before assembly. Double-check pinouts on any modules you purchase as they may vary by manufacturer.
