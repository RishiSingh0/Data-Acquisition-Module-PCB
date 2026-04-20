# Data Acquisition Module PCB

6-layer, 50mm x 50mm PCB designed to collect and log real-time sensor data aboard a human-powered submarine.

**Status: V1 archived.** This is the first spin of the SUBC data acquisition module. It's kept here for reference. V2 is a ground-up redesign, not a rework of this board; see the V2 section below for what changed and why.

![Fully assembled PCB front side, Prototype V1](assets/front.png)

## Overview

This board serves as the central data acquisition system for a human-powered submarine. It collects environmental sensor data, provides pilot feedback via LEDs, and logs everything to an onboard SD card.

**Key capabilities:**
- Barometric pressure and depth measurement
- Temperature and humidity monitoring
- Propeller RPM tracking (via external tachometer interface)
- Onboard SD card data logging
- LED status indicators for pilot feedback
- Bluetooth-ready antenna placement for future connectivity

## Hardware

| Spec | Detail |
|------|--------|
| MCU | BT840F (Nordic nRF52-based) |
| Board size | 50mm x 50mm |
| Layer count | 6 |
| Sensors | Barometer (depth/pressure), temperature/humidity |
| Communication | I2C (local), UART + LVDS (long-distance) |
| Storage | Micro SD card (bottom-mounted) |
| Feedback | LED system (5V, via cascaded logic level shifters) |
| Manufacturer | JLCPCB |

## Schematic

![Circuit schematic](assets/Schematic.png)

## Design Evolution

The board went through three design iterations to address signal integrity and routing constraints:

### 2-Layer (Initial)
The initial design used a standard 2-layer stackup. The high number of traces and high-speed signal requirements quickly exceeded what two layers could reliably support.

### 4-Layer (Revised)
A four-layer board improved routing flexibility, but concerns remained about signal isolation, particularly with numerous high-speed signals routed to surface-mount pads and potential trace interference.

### 6-Layer (Final)
The final 6-layer design provides proper ground plane isolation between signal layers, adequate routing channels around constrained components (antenna, SD card, connectors), and reliable high-speed signal integrity.

## Layout and Constraints

The 50mm x 50mm board dimensions were constrained by the submarine enclosure. Two components imposed strict keep-out zones:

- **Antenna (top layer):** No copper or components beneath it to ensure Bluetooth RF performance
- **SD card (bottom layer):** Clear area underneath required for card insertion and mechanical clearance

Connectors were pre-positioned to align geographically with the power board, further constraining component placement. The 6-layer stackup was essential to route around these restrictions.

### PCB, Pre-Assembly

| Front | Back |
|-------|------|
| ![PCB front side](assets/frontb.png) | ![PCB back side](assets/backb.png) |

### Prototype V1, Assembled

| Front | Back |
|-------|------|
| ![Assembled front](assets/front.png) | ![Assembled back](assets/back.png) |

## Challenges and Solutions

### Long-Distance I2C Failure
**Problem:** The sensor I2C bus was unreliable over long cable runs. Cable capacitance degraded signal transitions, causing communication errors.

**Solution:** A local microcontroller reads sensor data via a short I2C connection, then transmits over the long cable using UART paired with LVDS (Low-Voltage Differential Signaling). LVDS sends a signal and its inverse on a wire pair, canceling out external noise and enabling reliable long-distance communication.

### LED Voltage Mismatch
**Problem:** The MCU outputs 1.6V logic, but the LED system requires 5V. The existing 3.3V-to-5V logic level shifter cannot handle 1.6V input.

**Solution:** Two cascaded logic level shifters: 1.6V to 3.3V, then 3.3V to 5V. Since LED control is one-way (no return data), the cascaded topology introduces no timing concerns.

### BGA Soldering
**Problem:** The BT840F MCU uses a Ball Grid Array package, which is extremely difficult to hand-solder. Initial hand-soldering attempts were unreliable and time-consuming.

**Solution:** A solder paste stencil was ordered to improve consistency. Professional assembly service (JLCPCB) is also being evaluated to eliminate manual soldering errors entirely.

### JTAG Programming Failure (Error -102)
**Problem:** The prototype failed to program via JTAG. The J-Link reported error -102, which means the programmer couldn't reliably communicate with the chip. That points at a hardware connectivity issue rather than a firmware or device defect.

**Root cause:** CLK and DIO lines were swapped in the schematic, which carried through into the layout. V1 was not reworked to swap them back. The finding carried forward into V2, which moves to SWD and cross-checks the debug pinout against the programmer reference on both the schematic and the layout before fab.

See the full [JTAG Debug Report](docs/JTAG_Debug_Report.md) for the debug trail.

## Project Status

### Prototype V1 (archived)

- [x] Schematic and 6-layer PCB layout
- [x] Fabrication (JLCPCB) and component sourcing
- [x] Hand assembly with solder paste stencil, including the BGA MCU
- [x] Bring-up to the JTAG stage, root-caused to the CLK/DIO pin swap

V1 never got to a working blinky. The JTAG pinout error, combined with a broader rethink of how power should live on this board (see V2), made a rework unattractive. V1 was closed out here as a prototype learning exercise and the findings rolled forward into V2.

### V2 Redesign

V2 is a ground-up redesign, not a patch of V1. Three things from V1 drive the new design, and the full write-up lives in the V2 repo:

1. **JTAG pinout error.** V2 moves to SWD, which drops the pin count, and the debug pinout gets cross-checked against the programmer reference on both the schematic and the layout before fab.

2. **Power was off-board in V1.** A separate analog power board fed pre-regulated rails into the DAQ through the external connectors, so every rail on this logic board came in over an inter-board connector and the board had no power intelligence of its own. V2 consolidates power onto the motherboard around Nordic's **nPM1300** PMIC, which packs USB charging, battery power path, system rails, and fuel gauging into one IC designed to pair with the nRF52. USB-C goes on the motherboard directly.

3. **Hall-effect sensing was on the motherboard in V1.** Running an analog sensing chain next to a switching logic board is fine on its own, but once the sub is integrated, motor and power-stage noise couples into that chain. V2 spins the hall-effect sensing out onto a dedicated daughter board with its own op-amp front end and a 4–20 mA current loop for noise-immune long cable runs, so the sensing path is physically and electrically isolated from the logic board.

Alongside those three, V2 also picks up USB for power and data, a UART debug interface, NAND flash for expanded logging, EEPROM for calibration storage, and sensors integrated directly on the board.

## Repository Structure

```
Data-Acquisition-Module-PCB/
├── README.md
├── assets/
│   ├── Schematic.png          # Circuit schematic
│   ├── front.png              # Assembled PCB front (V1)
│   ├── back.png               # Assembled PCB back (V1)
│   ├── frontb.png             # Bare PCB front
│   ├── backb.png              # Bare PCB back
│   ├── closeup_debug.png      # Debug session close-up
│   └── connection_setup.png   # JTAG debug setup
└── docs/
    └── JTAG_Debug_Report.md   # Error -102 investigation
```
