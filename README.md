# 🚦 Smart 4-Way Traffic Light Controller

![Platform](https://img.shields.io/badge/Platform-ATmega32-blue?style=for-the-badge&logo=atmel)
![Language](https://img.shields.io/badge/Language-C%20%28AVR--GCC%29-green?style=for-the-badge&logo=c)
![Sensor](https://img.shields.io/badge/Sensor-DHT11-orange?style=for-the-badge)
![Simulator](https://img.shields.io/badge/Simulator-Proteus-red?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

> A fully embedded smart traffic light system built on **ATmega32**, featuring dual-direction countdown timers (7-segment), a DHT11 temperature & humidity sensor, and a 16×2 LCD displaying real-time safety messages — all programmed in bare-metal C with direct register manipulation.

---

## 📸 Circuit Simulation (Proteus)

![Proteus Simulation](Smart_traffic_light.png)

---

## ✨ Features

| Feature | Details |
|---|---|
| 🚗 **Dual-Direction Control** | Two independent traffic directions (Direction 1 & Direction 2) |
| ⏱️ **Countdown Timer** | 40-second countdown displayed on dual 7-segment displays (tens + ones) |
| 🌡️ **DHT11 Sensor** | Real-time temperature & humidity reading every 2 seconds |
| 📟 **LCD Safety Messages** | Dynamic safety tips based on remaining time |
| 🟡 **Yellow Blink Warning** | Green blinks at `t ≤ 6s`, Yellow activates at `t ≤ 3s` |
| ⚙️ **No HAL / No Delay Timers** | Pure register-level C — no Arduino, no HAL library |

---

## 🔧 Hardware Components

- **Microcontroller:** ATmega32 @ 8 MHz
- **Sensor:** DHT11 (Temperature & Humidity) → `PA7`
- **Display:** 16×2 LCD in **4-bit mode** → `PORTC` (data) + `PD3/PD4/PD7` (control)
- **7-Segment Displays:** BCD input via `7447` decoder
  - Tens digit → `PORTA [3:0]`
  - Ones digit → `PORTC [3:0]`
- **Traffic LEDs:**

| Signal | Pin |
|---|---|
| GREEN_1 | PB0 |
| YELLOW_1 | PB1 |
| RED_1 | PB2 |
| GREEN_2 | PB3 |
| YELLOW_2 | PB4 |
| RED_2 | PB5 |

---

## 🗂️ Project Structure

```
SmartTrafficLight/
├── main.c               # Main application logic & traffic state machine
├── DHT.c / DHT.h        # DHT11 driver (bit-banging protocol)
├── DHT_Settings.h       # DHT pin configuration
├── lcd.c / lcd.h        # LCD driver (4-bit & 8-bit modes)
├── lcd_config.h         # LCD pin mapping
├── Common_Macros.h      # Bit manipulation macros (Set/Clear/Toggle)
├── IO_Macros.h          # High-level GPIO macros (PinMode, DigitalWrite...)
├── Smart_traffic_light.pdsprj  # Proteus simulation project
└── README.md
```

---

## ⚙️ How It Works

The system runs in an infinite loop alternating between two traffic directions:

```
Direction 1 GREEN  ──►  40s countdown  ──►  Blink  ──►  Yellow  ──►  RED
Direction 2 GREEN  ──►  40s countdown  ──►  Blink  ──►  Yellow  ──►  RED
     └────────────────────────── repeat forever ──────────────────────────┘
```

### 🕹️ Traffic Light Logic (per direction)

```c
if (t > 6)       → GREEN solid ON        (1 second per step)
else if (t > 3)  → GREEN blinks          (250ms ON / 250ms OFF)
else             → GREEN OFF, YELLOW ON  (warning phase)
```

### 📟 LCD Safety Messages

| Time Remaining | Message |
|---|---|
| `t > 30s` | `Drive Safely!` |
| `t > 20s` | `Speed Kills!` |
| `t > 10s` | `Fasten Seatbelt!` |
| `t ≤ 10s` | `Keep Your Lane` |

> Second row always shows: `T:25C | H:60%`

---

## 🔌 Pin Configuration Summary

```
ATmega32
├── PORTA [3:0]  →  7-Segment TENS digit (BCD → 7447)
├── PORTB [5:0]  →  Traffic LEDs (R/Y/G × 2 directions)
├── PORTC [3:0]  →  7-Segment ONES digit (BCD → 7447)
├── PORTC [7:4]  →  LCD Data lines (D4–D7, 4-bit mode)
├── PORTD [3]    →  LCD RS
├── PORTD [4]    →  LCD RW
├── PORTD [7]    →  LCD EN
└── PORTA [7]    →  DHT11 Data line
```

---

## 🛠️ Build & Flash

### Prerequisites
- **Atmel Studio 7** or **AVR-GCC** toolchain
- **avrdude** for flashing
- **Proteus 8+** for simulation

### Compile with AVR-GCC

```bash
avr-gcc -mmcu=atmega32 -DF_CPU=8000000UL -O1 \
    main.c DHT.c lcd.c -o traffic.elf

avr-objcopy -O ihex traffic.elf traffic.hex
```

### Flash to Hardware

```bash
avrdude -c usbasp -p m32 -U flash:w:traffic.hex
```

### Simulate in Proteus
Open `Smart_traffic_light.pdsprj` in Proteus and press **Play ▶**

---

## 📚 Key Libraries / Drivers

### `DHT.c` — Bit-Bang DHT11 Driver
- Sends 20ms LOW start signal
- Reads 40-bit frame: `RH_int | RH_dec | T_int | T_dec | Checksum`
- Verifies checksum before updating values

### `lcd.c` — 4-bit LCD Driver
- Supports both 8-bit and 4-bit modes
- `lcd_set_cursor(row, col)` for precise positioning
- `lcd_write_string()` for string output

### `Common_Macros.h` — Bit Macros
```c
Set_Bit(REG, BIT)
Clear_Bit(REG, BIT)
Toggle_Bit(REG, BIT)
Bit_Is_Set(REG, BIT)
```

### `IO_Macros.h` — GPIO Abstraction
```c
PinMode(B, 0, Output);
DigitalWrite(B, 0, High);
DigitalRead(B, 0);
```

---


> **Course:** EEC 214 — Embedded Systems  
> **Supervisor:** Dr. Mohamed Torad  
> **Institute:** Higher Technological Institute (H.T.I.) — Group 2

---

## 📄 License

This project is licensed under the **MIT License** — feel free to use, modify, and distribute with attribution.

---

<div align="center">
  <sub>Built with ❤️ using bare-metal AVR-C | No HAL, No Arduino, Just Registers.</sub>
</div>
