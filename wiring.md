# Wiring Guide

This document covers all physical wiring for the PiGate build, including the TFT display, cooling fan, and GPIO header layout.

---

## Raspberry Pi 3B+ GPIO Header

The Pi's 40-pin GPIO header is arranged in two rows running the length of the board. Pin 1 (3.3V) is located at the corner closest to the SD card slot.

```
SD card end
[2][4][6][8][10][12][14][16][18][20][22][24][26][28][30][32][34][36][38][40]  <- outer row (even)
[1][3][5][7][ 9][11][13][15][17][19][21][23][25][27][29][31][33][35][37][39]  <- inner row (odd)
```

- **Odd pins (1, 3, 5...)** — inner row, left to right
- **Even pins (2, 4, 6...)** — outer row, left to right
- Pin 1 and Pin 2 are immediate neighbors at the SD card end — take care not to confuse 3.3V (pin 1) with 5V (pin 2)

---

## 2.8" TFT SPI Display (ILI9341)

**Connection method:** Female-to-female Dupont jumper wires (2.54mm pitch), one wire per pin.

**Pins used:** 9 of the 14 available pins. Touch (CTP) and SD card pins are left disconnected as they are not used in this build.

| LCD Pin | Label | Function | Pi Physical Pin | Pi BCM GPIO |
|---|---|---|---|---|
| 1 | VCC | Power (3.3V) | 1 | 3.3V |
| 2 | GND | Ground | 6 | GND |
| 3 | LCD_CS | SPI chip select | 24 | GPIO8 (CE0) |
| 4 | LCD_RST | Display reset | 13 | GPIO27 |
| 5 | LCD_RS | Data/command select | 22 | GPIO25 |
| 6 | SDI/MOSI | SPI data out | 19 | GPIO10 |
| 7 | SCK | SPI clock | 23 | GPIO11 |
| 8 | LED | Backlight | 17 | 3.3V (always-on) |
| 9 | SDO/MISO | SPI data in | 21 | GPIO9 |

**Disconnected pins (not used):**

| LCD Pin | Label | Reason |
|---|---|---|
| 10 | CTP_SCL | Capacitive touch — not used |
| 11 | CTP_RST | Capacitive touch — not used |
| 12 | CTP_SDA | Capacitive touch — not used |
| 13 | CTP_INT | Capacitive touch — not used |
| 14 | SD_CS | Onboard SD slot — not used |

**Notes:**
- VCC must connect to **3.3V (pin 1)**, not 5V (pin 2) — connecting to 5V will damage the display
- LED is wired directly to 3.3V for always-on backlight — no software backlight control needed
- SPI must be enabled via `raspi-config` before the display will be detected

---

## Cooling Fan

**Type:** 5V 2-pin fan (red/black wires, always-on)

| Fan Wire | Color | Pi Physical Pin |
|---|---|---|
| Power | Red | 2 (5V) |
| Ground | Black | 14 (GND) |

**Notes:**
- Fan runs continuously whenever the Pi is powered on — no software control required
- Any available GND pin can be used if pin 14 is occupied

---

## Full GPIO Pin Usage Summary

| Pi Physical Pin | Assignment |
|---|---|
| 1 (3.3V) | LCD VCC |
| 2 (5V) | Fan power |
| 6 (GND) | LCD GND |
| 13 (GPIO27) | LCD RST |
| 14 (GND) | Fan GND |
| 17 (3.3V) | LCD LED (backlight) |
| 19 (GPIO10) | LCD MOSI |
| 21 (GPIO9) | LCD MISO |
| 22 (GPIO25) | LCD RS (DC) |
| 23 (GPIO11) | LCD SCK |
| 24 (GPIO8) | LCD CS |
| *remaining* | Available for future use |
