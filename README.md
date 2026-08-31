# Krakenbane One — Rocketry DAQ Flight Computer

A data acquisition (DAQ) flight computer for a model rocket, designed for the Launch Canada competition. The system is two KiCad projects: a main flight computer board and a GNSS breakout that mounts separately and connects over a cable.

This is my first PCB design using SMT components, developed with support from university rocketry team members.

## Boards

|         | [DAQ-V4.1](DAQ-V4.1/) | [DAQ-GPS_Breakout-V1.0](DAQ-GPS_Breakout-V1.0/) |
| ------- | --------------------- | ----------------------------------------------- |
| Role    | Main flight computer  | GNSS receiver, mounted remotely                 |
| Outline | 38 × 100 mm           | 42 × 70 mm                                      |
| Stackup | 4-layer FR4, ~1.6 mm  | 4-layer FR4, ~1.6 mm                            |

The two boards are joined by a 6-pin JST GH (1.25 mm) cable carrying 3.3 V, GND, UART TX/RX, `TIMEPULSE`, and `NRESET`.

## Main board (DAQ-V4.1)

**MCU** — STM32F407VGT6 (Cortex-M4F, LQFP-100), 16 MHz crystal (NX3225GA), SWD debug on a 2×5 1.27 mm header with SWO.

**Sensors** — each on its own I²C bus, so there are no address collisions and each part can run at its full rate:

| Part          | Function                                | Range        |
| ------------- | --------------------------------------- | ------------ |
| LSM6DSV32X    | 6-axis IMU (accel + gyro), `INT` to MCU | ±32 g        |
| ADXL375       | High-g accelerometer                    | ±200 g       |
| MS5607-02BA03 | Barometric pressure / altitude          | 10–1200 mbar |

**Storage**

- microSD card (Hirose DM3BT push-pull socket) on **SPI1**, with 33 Ω series resistors on the clock and data lines, 4.7 kΩ bus pull-ups, and card-detect wired to `PE2`.
- W25Q64JV 64 Mbit NOR flash on **SPI2** for onboard logging independent of the card.

**Radio / telemetry** — XBee module on a 6-pin JST GH connector, driven from **USART2** with RTS/CTS hardware flow control.

**Power** — XT30 input at ~7.5 V → SS34 reverse-polarity diode → AP63203 synchronous buck to 3.3 V (SPM5030T 3.3 µH inductor). A separate `+3.3VA` analog rail is filtered from the digital rail through a ferrite bead for the MCU's analog supply. Power and status LEDs on board.

**Audio** — CMT-0502 magnetic buzzer switched by an AO3400A MOSFET with a B5819W flyback diode, for arming and continuity tones.

## GNSS breakout (DAQ-GPS_Breakout-V1.0)

u-blox SAM-M10Q with integrated patch antenna, broken out to a horizontal JST GH connector. Placed on its own board so the antenna can sit away from the flight computer's switching regulator and radio, with a clean ground plane underneath.

## Repository layout

```
DAQ-V4.1/                    Main flight computer
  DAQ-V4.1.kicad_sch/_pcb    Schematic and layout
  DAQ-V4.1.pdf               Schematic export
  DAQ-V4-*.pdf               Per-layer plots (copper, mask, paste, silkscreen, edge cuts)
  production/                Gerbers, BOM, pick-and-place, IPC netlist
  DRC.rpt / ERC.rpt          Latest design and electrical rule check reports
  *-backups/                 KiCad autosave history

DAQ-GPS_Breakout-V1.0/       GNSS breakout, same structure
```

Production files are generated with the KiCad Fabrication Toolkit and are formatted for JLCPCB assembly (`bom.csv` carries LCSC part numbers, `positions.csv` the placement data).

## Opening the project

Requires KiCad 8 or newer. Open `DAQ-V4.1/DAQ-V4.1.kicad_pro` or `DAQ-GPS_Breakout-V1.0/DAQ-GPS_Breakout-V1.0.kicad_pro`. Several footprints and 3D models come from project-local libraries, so open the project file rather than the `.kicad_pcb` directly.
