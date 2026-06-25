# Airplane Navigation & Telemetry Board

A compact avionics board built around a bare **STM32F411RET6** chip microcontroller, integrating GPS, an inertial measurement unit (IMU), and a barometric pressure sensor on a single I²C bus. Designed as a self-contained navigation and telemetry computer for a fixed-wing aircraft, with onboard power regulation and SWD debug access.

## 3D layout
<img width="386" height="437" alt="Screenshot 2026-06-24 180325" src="https://github.com/user-attachments/assets/c9760df5-4d9a-4ea4-ba8b-05ea4e0a1870" />

---

## 2D layout
<img width="369" height="419" alt="Screenshot 2026-06-24 180250" src="https://github.com/user-attachments/assets/a1b7ff09-d18a-4f01-96fe-493a35522b96" />

---

## PCB Schematic
<img width="717" height="409" alt="Screenshot 2026-06-24 180141" src="https://github.com/user-attachments/assets/9f0ee447-fd40-459f-9dc0-ff70aa21bfb1" />

---

## I2C BUS
<img width="670" height="382" alt="Screenshot 2026-06-12 092646" src="https://github.com/user-attachments/assets/d727bd87-3084-4534-bdb8-3b3906659922" />

---

## GPS
<img width="686" height="395" alt="Screenshot 2026-06-24 180219" src="https://github.com/user-attachments/assets/5ab7d681-c850-48df-9bc0-06d5f7f7de4a" />



---

## Overview

This board consolidates the flight controller and the sensor suite onto one PCB. Rather than feeding a separate autopilot, the onboard STM32F411 reads all sensors directly and handles navigation and telemetry itself. Everything positional and inertial sits on one shared I²C bus, keeping the design simple and the routing tight.

It is the successor to a previous module-based sensor board; the architecture here moves the microcontroller onto the board (bare chip rather than a dev module) and switches the GPS to an I²C interface.

> Predecessor project: `<link to previous PCB repository>`

---

## Features

- **STM32F411RET6** Cortex-M4F MCU (LQFP64, up to 100 MHz, 512 KB flash) — bare chip with full support circuitry
- **GPS**, **6-axis IMU**, and **barometer** on a single shared I²C bus
- Onboard **5 V → 3.3 V** regulation with reverse-polarity protection
- **5-pin SWD** debug/programming header
- **4-layer** stackup with a solid internal ground plane for clean signal return and low EMI
- Hierarchical schematic organized by functional block

---

## Hardware Architecture

### Microcontroller
- **STM32F411RET6**, LQFP64 package
- Full support circuit: per-pin VDD decoupling, VCAP regulator capacitor, filtered VDDA, NRST cap, BOOT0 pull-down, VBAT tied to rail
- Internal HSI clock (no external crystal required for the current bus speeds)

### Sensors (I²C bus)
| Device        | Function              | Interface | Part                    |
|---------------|-----------------------|-----------|-------------------------|
| GPS           | Position / time       | I²C       | NEO-M9N-00B             |
| MPU6050       | 6-axis IMU            | I²C       | InvenSense MPU6050      |
| BMP280        | Barometric pressure   | I²C       | Bosch BMP280            |
| Tof tracker   | Time of flight        | I²C       | VL53L0X                 |

All devices share **SCL/SDA on PB6/PB7 (I²C1)** with a single pair of pull-up resistors for the whole bus. Confirm each device's address is unique before bring-up.

### Power
- Input: **5 V** from the aircraft BEC rail via a keyed 2-pin connector
- Regulator: **LT1763** low-noise LDO → **3.3 V**
- Input bulk + decoupling capacitors and reverse-polarity protection at the connector

### Debug / Programming
- **SWD header (5-pin):** SWDIO, SWCLK, GND, 3V3, NRST
- NRST broken out to enable connect-under-reset recovery
- Programmed with an external ST-Link (or compatible probe)

---

## Repository Structure

```
.
├── Airplane_Navigation_&_Telemetry_Board.PrjPcb     # Altium project
├── Airplane_Navigation_&_Telemetry_Schematic.SchDoc # Top sheet (MCU + power)
├── I2C_BUS.SchDoc                                   # I²C sensor sub-sheet
├── GPS.SchDoc                                        # GPS sub-sheet
├── Airplane_System_Board.PcbDoc                      # PCB layout
├── Airplane_Navigation_&_Telemetry_Board.BomDoc      # Bill of materials
└── README.md
```

---

## Design Notes

- **Schematic hierarchy:** power and ground use global power-port symbols; cross-sheet signals (SDA, SCL) use ports/sheet entries.
- **Decoupling:** each VDD pin has its own 100 nF placed at the pin, over a shared bulk capacitor; VCAP carries a single 4.7 µF; VDDA is decoupled with 100 nF + 1 µF.
- **Stackup:** signal / ground plane / power / signal — keep the ground plane under the top signal layer unbroken.
- **Placement:** MCU central as the hub, sensors clustered for a short I²C bus, power section at the input edge, connectors on the board edge.

---

## Status

| Item              | State            |
|-------------------|------------------|
| Schematic         | In progress      |
| PCB layout        | In progress      |
| Fabrication       | Not yet ordered  |
| Firmware          | Not started      |

---

## Tools

- **Altium Designer** (schematic capture + PCB layout)
- **ST-Link** + STM32CubeMX / CubeIDE for firmware and pin planning

