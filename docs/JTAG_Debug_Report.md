# BT840F Prototype PCB — JTAG Debug Report

**Date:** February 28, 2026
**Target Device:** Nordic nRF52 (BT840F MCU)
**Issue:** J-Link JTAG Programming Failure (Error -102)

---

## Summary

The BT840F prototype PCB failed to program via the JTAG interface during firmware flashing. The `nrfjprog` tool reported error -102, indicating a communication failure between the J-Link programmer and the target device.

---

## Test Setup

| Component | Detail |
|-----------|--------|
| MCU | BT840F (minimal prototype — MCU, JTAG, crystal only) |
| Programmer | J-Link via JTAG (dev board P19 connector) |
| Power | 3.3V, 0.5A via jumper cables to 3.3V and GND pads |
| Firmware | Blinky app (verified on dev board), blank code flashed to prototype |
| Tool | `west flash` with `nrfjprog` runner |

### Connection Setup
![JTAG connection setup between dev board and prototype PCB](../assets/connection_setup.png)

### PCB Close-Up
![Close-up of prototype PCB showing BT840F MCU and JTAG pads](../assets/closeup_debug.png)

---

## Error Output

```
-- west flash: using runner nrfjprog
-- runners.nrfjprog: Flashing file: /path/to/blinky/build/zephyr/zephyr.hex

[error] [ Client] - Encountered error -102: Command read_device_info
         executed for 130 milliseconds with result -102
[error] [ Worker] - An unknown error.
[error] [ Client] - Encountered error -102: Command read_memory_descriptors
         executed for 24 milliseconds with result -102
Failed to read device memories.

ERROR: JLinkARM DLL reported an error.
FATAL ERROR: command exited with status 33
```

**Error -102** means the J-Link cannot reliably read or write to the chip — this is a **hardware connectivity issue**, not a firmware or device defect.

---

## Root Cause Analysis

### 1. Connection Quality
- Poor solder joints on JTAG pads
- Loose or damaged connector pins
- Intermittent contact in JTAG cable

### 2. Signal Integrity
- Inadequate trace impedance matching for JTAG lines
- Missing or insufficient pull-up/pull-down resistors
- Noise coupling into JTAG signal lines

### 3. Power Supply
- Voltage brownout during JTAG communication
- 0.5A current may be marginal for the MCU + programmer
- Power supply ripple affecting MCU stability

---

## Recommended Actions

### Hardware Verification
1. Inspect all JTAG solder joints under magnification for cold joints, bridges, or cracked traces
2. Verify continuity on all JTAG pins (TDI, TDO, TMS, TCK, RESET) with multimeter
3. Measure 3.3V supply at MCU pins under load
4. Monitor supply voltage during JTAG operations with oscilloscope
5. Try alternative JTAG cable and reflow connector pads

### PCB Design Review
1. Verify JTAG trace impedance matches J-Link specifications
2. Confirm pull-up/pull-down resistor values per Nordic hardware design guide
3. Review power delivery network decoupling near MCU
4. Verify crystal load capacitor values

---

## Status

**Blocked** — JTAG connectivity must be verified before sending revision to JLCPCB for manufacturing.
