# Custom Keyboard PCB

A custom mechanical keyboard PCB built around a **Teensy 4.0**, with a **16 x 6 key matrix**, a small **ST7789 color display** below the keys, and a **rotary encoder with a push-button**.

> **Status:** work in progress. Pin assignments are planned; schematic, PCB layout, and firmware are still to come.

## Features

- 16 columns x 6 rows key matrix (up to 96 key positions), one diode per switch
- 1.14" IPS color display (ST7789, 135 x 240) mounted below the keys
- Rotary encoder with push-button
- Teensy 4.0 as the microcontroller (native USB)
- MCP23017 I2C I/O expander for the 16 column lines

## Hardware

| Part | Notes |
|---|---|
| Teensy 4.0 | Main MCU. 3.3 V logic only (not 5 V tolerant). Micro-USB connector. |
| MCP23017 | 16-bit I2C expander. Reads the 16 matrix columns. Address 0x20 (A0-A2 tied to GND). |
| ST7789 1.14" IPS module | 135 x 240, 8-pin SPI (GND, VCC, SCL, SDA, RES, DC, CS, BLK), 3.3 V. |
| Rotary encoder (EC11 style) | Quadrature outputs plus push-button. |
| Switch diodes | One per key, anode toward the switch / column side, cathode (stripe) toward the row. |
| Passives | 2.2 kOhm I2C pull-ups (SDA, SCL), 0.1 uF decoupling on the MCP23017 VDD. |

## Why an I/O expander?

The design needs 22 matrix lines, about 6 display lines, and 3 encoder lines, which is 30 GPIO in total. The Teensy 4.0's through-hole edge pins only provide 24 (pins 0-23). Putting the 16 columns on an MCP23017 keeps everything on the standard through-hole footprint, and a full 6-row scan stays well under a millisecond at 400 kHz I2C.

## Pin assignments

Full details, including every Teensy header pin and every MCP23017 pin, are in a spreadsheet.

### Teensy 4.0

| Function | Teensy pin |
|---|---|
| Matrix rows ROW0-ROW5 | 0, 1, 2, 3, 4, 5 |
| Encoder A / B | 6 / 7 |
| Encoder button | 14 |
| Display RES / DC / CS | 8 / 9 / 10 |
| Display SDA (MOSI) / SCL (SCK) | 11 / 13 |
| Display backlight (BLK, PWM) | 15 |
| I2C SDA / SCL (to MCP23017) | 18 / 19 |
| MCP23017 interrupt (optional) | 20 |
| Spare | 12, 16, 17, 21, 22, 23 |

### MCP23017 (28-pin SPDIP / SOIC / SSOP)

| Matrix columns | MCP23017 port | Chip pins |
|---|---|---|
| COL0-COL7 | GPA0-GPA7 | 21-28 |
| COL8-COL15 | GPB0-GPB7 | 1-8 |

Other connections: VDD to 3V3, VSS to GND, A0-A2 to GND, RESET to 3V3, SCL to Teensy 19, SDA to Teensy 18, INTA to Teensy 20 (optional).

## How the matrix scan works

The Teensy keeps all six row pins as inputs (high-Z). To scan a row, it drives that row LOW, reads both MCP23017 ports in a single I2C transaction (columns have internal pull-ups enabled), then releases the row. A LOW column bit means the key at that row and column is pressed.

## Firmware

Firmware is planned for the Arduino / Teensyduino toolchain. Libraries likely to be used:

- An ST7789 driver such as `Adafruit_ST7789` with `Adafruit_GFX`, or the Teensy-optimized `ST7789_t3`
- `Encoder` for the rotary encoder
- An MCP23017 library, or direct register access over `Wire`

The ST7789 135 x 240 panel needs a window offset because the controller's memory is 240 x 320. If an edge of the screen shows a garbage strip, check the width, height, and offset in the driver setup.

## Notes

- Teensy I/O is 3.3 V only. Keep every connected device at 3.3 V logic.
- On pin 13, the SPI clock shares the Teensy's onboard LED, which will flicker faintly during display updates. This is harmless.
- Verify the display module's pin order against its silkscreen, as vendors differ.
