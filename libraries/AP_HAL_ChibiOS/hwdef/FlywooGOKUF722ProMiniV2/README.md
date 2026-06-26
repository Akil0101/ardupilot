# Flywoo GOKU F722 PRO MINI V2 — ArduPilot Board Definition

20×20 mm flight controller based on STM32F722RET6.

---

## Hardware Map

### MCU
| Item | Value |
|------|-------|
| Chip | STM32F722RET6 |
| Core | Cortex-M7 @ 216 MHz |
| Flash | 512 KB (sectors 0–7) |
| RAM | 256 KB |
| FPU | DP FPU (double precision) |
| USB | Full-speed (PA11/PA12) |

### IMU — ICM42688P
| Signal | Pin | Notes |
|--------|-----|-------|
| SPI bus | SPI1 | |
| SCK | PA5 | |
| MISO | PA6 | |
| MOSI | PA7 | |
| CS | PA4 | Active low |
| INT | PC3 | EXTI interrupt |
| Rotation | CW270 | `ROTATION_YAW_270` |

Second gyro footprint (CS=PB2, INT=PC4) exists on PCB but is unpopulated on V2.

### Barometer — DPS310
| Signal | Value |
|--------|-------|
| Bus | I2C1 |
| SCL | PB8 |
| SDA | PB9 |
| Address | 0x77 |

### OSD — MAX7456 / AT7456E
| Signal | Pin |
|--------|-----|
| SPI bus | SPI2 |
| SCK | PB13 |
| MISO | PB14 |
| MOSI | PB15 |
| CS | PB12 |

### Blackbox Flash — W25Q128FV (16 MB)
| Signal | Pin |
|--------|-----|
| SPI bus | SPI3 |
| SCK | PC10 |
| MISO | PC11 |
| MOSI | PB5 |
| CS | PC13 |
| JEDEC ID | EF 40 18 |

### UART Assignments
| Port | SERIAL# | TX | RX | Default role |
|------|---------|-----|-----|--------------|
| USB | SERIAL0 | PA11 | PA12 | MAVLink (Mission Planner) |
| USART1 | SERIAL1 | PA9 | PA10 | RC input (SBUS / ELRS) |
| USART2 | SERIAL2 | PA2 | PA3 | GPS |
| USART3 | SERIAL3 | PB10 | PB11 | Telemetry (MAVLink) |
| UART4 | SERIAL4 | PA0 | PA1 | ESC telemetry |
| UART5 | SERIAL5 | PC12 | PD2 | SmartAudio / VTX |
| USART6 | SERIAL6 | PC6 | PC7 | MSP / DJI OSD |

### Motor Outputs
| Pad | Pin | Timer | CH | DMA |
|-----|-----|-------|----|-----|
| M1 | PB1 | TIM3 | CH4 | DMA1 S2 Ch5 |
| M2 | PB4 | TIM3 | CH1 | DMA1 S4 Ch5 |
| M3 | PB3 | TIM2 | CH2 | DMA1 S6 Ch3 |
| M4 | PA15 | TIM2 | CH1 | DMA1 S5 Ch3 |
| M5 | PC8 | TIM8 | CH3 | DMA2 S4 Ch7 |
| M6 | PC9 | TIM8 | CH4 | DMA2 S7 Ch7 |
| M7 | PB6 | TIM4 | CH1 | DMA1 S0 Ch2 |
| M8 | PB7 | TIM4 | CH2 | DMA1 S3 Ch2 |

All motors have unique DMA streams — bidirectional DShot supported on all 8 outputs.

### LED Strip
| Signal | Pin | Timer |
|--------|-----|-------|
| WS2812B data | PA8 | TIM1 CH1, DMA2 S6 Ch0 |

### ADC
| Function | Pin | ADC Channel | GPIO# |
|----------|-----|-------------|-------|
| VBAT | PC1 | ADC3 IN11 | 12 |
| CURR | PC0 | ADC3 IN10 | 13 |
| RSSI | PC2 | ADC3 IN12 | 14 |

Voltage divider: 10 kΩ + 1 kΩ → 11:1. Calibrate `BATT_VOLT_MULT` before first flight.
Current: ibata_scale=170 → ArduPilot `BATT_AMP_PERVLT` = 17.0 A/V.

### GPIO
| Function | Pin | GPIO# |
|----------|-----|-------|
| Status LED | PC15 | 0 |
| Beeper | PC14 | 80 |
| VTX power (PINIO1) | PB0 | 81 |

---

## Rotations

The ICM42688P is mounted at 270° clockwise relative to the forward arrow:

```
ArduPilot IMU rotation: ROTATION_YAW_270
```

Verify with Mission Planner's IMU calibration page — the X-axis must point forward.

---

## Flash Layout

```
0x08000000  Sector 0  16 KB  Bootloader
0x08004000  Sector 1  16 KB  Bootloader (cont.)
0x08008000  Sector 2  16 KB  EEPROM / Parameters
0x0800C000  Sector 3  16 KB  EEPROM / Parameters
0x08010000  Sector 4  64 KB  Application start
0x08020000  Sector 5 128 KB  Application
0x08040000  Sector 6 128 KB  Application
0x08060000  Sector 7 128 KB  Application
```

---

## Building

### Prerequisites
```bash
git clone --recursive https://github.com/ArduPilot/ardupilot.git
cd ardupilot
```

### Copy board files
```bash
cp -r /path/to/FlywooGOKUF722ProMiniV2 \
       libraries/AP_HAL_ChibiOS/hwdef/
```

### Register board ID
Add to `Tools/AP_Bootloader/AP_Bootloader.h`:
```c
#define AP_HW_FLYWOOF722PROMINI_V2 1051
```

### Build bootloader
```bash
./waf configure --board FlywooGOKUF722ProMiniV2 --bootloader
./waf bootloader
# Output: build/FlywooGOKUF722ProMiniV2/bin/AP_Bootloader.bin
```

### Build ArduCopter firmware
```bash
./waf configure --board FlywooGOKUF722ProMiniV2
./waf copter
# Output: build/FlywooGOKUF722ProMiniV2/bin/ArduCopter.apj
#         build/FlywooGOKUF722ProMiniV2/bin/ArduCopter.bin
#         build/FlywooGOKUF722ProMiniV2/bin/ArduCopter.hex
```

### Build other vehicle types
```bash
./waf plane      # ArduPlane
./waf rover      # ArduRover
./waf sub        # ArduSub
```

---

## Flashing

### Method 1: Mission Planner (recommended)
1. Open Mission Planner → Setup → Install Firmware.
2. Click **Load custom firmware** and select `ArduCopter.apj`.
3. Mission Planner will auto-detect the board via USB and flash.

### Method 2: STM32 DFU (board bricked / no bootloader)
1. Hold the boot button while connecting USB (enters ST ROM DFU mode).
2. Use STM32CubeProgrammer:
   - Connect: USB DFU
   - Erase full flash
   - Program `AP_Bootloader.bin` starting at `0x08000000`
   - Disconnect and reconnect USB (no boot button).
3. Flash ArduCopter via Mission Planner (step above).

### Method 3: dfu-util (Linux/macOS)
```bash
# Erase and flash bootloader
dfu-util -a 0 -s 0x08000000:leave -D AP_Bootloader.bin
# Then use Mission Planner for firmware
```

---

## Mission Planner Initial Setup

After flashing:

1. **Connect** USB → Mission Planner detects COM port, connect at 115200.
2. **Mandatory Hardware → Accel Calibration** — level calibration, then 6-position.
3. **Mandatory Hardware → Compass** — if external compass fitted on I2C1 pads.
4. **Mandatory Hardware → Radio Calibration**:
   - SBUS on SERIAL1: `SERIAL1_PROTOCOL=23`, `SERIAL1_BAUD=100`.
   - ELRS on SERIAL1: `SERIAL1_PROTOCOL=23`, `SERIAL1_BAUD=420`.
5. **Battery Monitor**: confirm `BATT_VOLT_MULT=11.0`, measure actual voltage and adjust.
6. **Motor Test** (with props off): verify motor order and direction.
7. **OSD**: `OSD_TYPE=1`, configure screens in the OSD tab.

---

## ELRS / CRSF Setup
```
SERIAL1_PROTOCOL 23
SERIAL1_BAUD 420
RC_PROTOCOLS 512
```

## DJI OSD (MSP)
```
SERIAL6_PROTOCOL 32
SERIAL6_BAUD 115
OSD_TYPE 5
```

## BiDir DShot + RPM Notch
```
MOT_PWM_TYPE 6
SERVO_BLH_BDMASK 255
INS_HNTCH_ENABLE 1
INS_HNTCH_MODE 5
INS_HNTCH_OPTS 2
```
