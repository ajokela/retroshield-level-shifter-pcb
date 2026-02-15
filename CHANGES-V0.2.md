# V0.2 Design Changes — KiCad Implementation Guide

This document describes all schematic and PCB changes required to update the RetroShield Level Shifter from V0.1 to V0.2. The BOM, CPL, and README have already been updated in the repository. This file captures the KiCad-specific changes that must be done in the KiCad GUI.

## Summary

Replace 5 of 9 TXB0108PW ICs with purpose-matched level translators to eliminate auto-direction-sensing failures on the Z80 bus. Add one DIR control line from the Arduino Giga to the new bidirectional data bus transceiver.

## Schematic Changes

### U1: TXB0108PW → SN74LVC541PWR (Address A0–A7)

- **Replace symbol** with 74LVC541 (TSSOP-20, same footprint size)
- **VCC** → 3.3V (was VCCA=3.3V / VCCB=5V on TXB0108)
- **GND** → GND
- **OE1_N, OE2_N** (pins 1, 19) → tie to GND (always enabled)
- **A1–A8** (pins 2–9) → Z80 address lines A0–A7 (5V side, 5V-tolerant inputs)
- **Y1–Y8** (pins 18–11) → Arduino Giga D22–D29 (3.3V side)
- **Remove R1** (10K pull-up resistor, was on TXB0108 OE pin)
- **Remove extra decoupling cap** — only 1 cap needed (VCC), not 2 (was VCCA + VCCB)

**Pin mapping (74LVC541):**
| Pin | Function | Net |
|-----|----------|-----|
| 1 | OE1_N | GND |
| 2 | A1 | ADDR_A0_5V |
| 3 | A2 | ADDR_A1_5V |
| 4 | A3 | ADDR_A2_5V |
| 5 | A4 | ADDR_A3_5V |
| 6 | A5 | ADDR_A4_5V |
| 7 | A6 | ADDR_A5_5V |
| 8 | A7 | ADDR_A6_5V |
| 9 | A8 | ADDR_A7_5V |
| 10 | GND | GND |
| 11 | Y8 | ADDR_A7_3V3 (D29) |
| 12 | Y7 | ADDR_A6_3V3 (D28) |
| 13 | Y6 | ADDR_A5_3V3 (D27) |
| 14 | Y5 | ADDR_A4_3V3 (D26) |
| 15 | Y4 | ADDR_A3_3V3 (D25) |
| 16 | Y3 | ADDR_A2_3V3 (D24) |
| 17 | Y2 | ADDR_A1_3V3 (D23) |
| 18 | Y1 | ADDR_A0_3V3 (D22) |
| 19 | OE2_N | GND |
| 20 | VCC | 3.3V |

### U2: TXB0108PW → SN74LVC541PWR (Address A8–A15)

- Same IC and wiring pattern as U1
- **A1–A8** → Z80 address lines A8–A15 (5V side)
- **Y1–Y8** → Arduino Giga D30–D37 (3.3V side)
- **Remove R2** (pull-up resistor)
- Reduce to 1 decoupling cap

### U3: TXB0108PW → SN74LVC541PWR (Control Inputs)

- Same IC as U1/U2
- **Only 4 of 8 channels used:**
  - A1 → MREQ_N_5V (pin D41)
  - A2 → IORQ_N_5V (pin D39)
  - A3 → RD_N_5V (pin D53)
  - A4 → WR_N_5V (pin D40)
  - A5–A8 → **tie to GND** (unused inputs must not float)
- **Y1–Y4** → corresponding 3.3V Arduino pins
- **Y5–Y8** → leave unconnected (outputs of unused channels)
- **Remove R3** (pull-up resistor)
- Reduce to 1 decoupling cap

### U4: TXB0108PW → SN74AHCT541PWR (Control Outputs)

- **Replace symbol** with 74AHCT541 (TSSOP-20, same footprint size)
- **VCC** → **5V** (not 3.3V — this drives 5V outputs)
- **OE1_N, OE2_N** → tie to GND
- **A1–A4** → Arduino 3.3V outputs (TTL-compatible inputs accept 3.3V HIGH):
  - A1 → CLK_3V3 (D38)
  - A2 → RESET_N_3V3 (D50)
  - A3 → INT_N_3V3 (D51)
  - A4 → NMI_N_3V3 (D52)
  - A5–A8 → **tie to GND** (unused inputs)
- **Y1–Y4** → Z80 control signals at 5V
- **Y5–Y8** → leave unconnected
- **Remove R4** (pull-up resistor)
- Reduce to 1 decoupling cap (VCC = 5V)

### U5: TXB0108PW → SN74LVC4245APWR (Data Bus)

- **Replace symbol** with SN74LVC4245A
- **Change footprint** from TSSOP-20 to **TSSOP-24** (4.4x7.8mm, 0.65mm pitch)
- **VCCA** (pin 1) → 3.3V
- **VCCB** (pin 24) → 5V
- **DIR** (pin 2) → new net "DATA_DIR" from Arduino D2 header pin
- **OE_N** (pin 23) → GND (always enabled)
- **A1–A8** (pins 3–6, 8–11) → Arduino data bus D42–D49 (3.3V side)
- **B1–B8** (pins 21–18, 16–13) → Z80 data bus D0–D7 (5V side)
- **GND** (pin 12) → GND
- **Remove R5** (pull-up resistor)
- **2 decoupling caps** needed (VCCA + VCCB)

**Pin mapping (SN74LVC4245A, TSSOP-24):**
| Pin | Function | Net |
|-----|----------|-----|
| 1 | VCCA | 3.3V |
| 2 | DIR | DATA_DIR (Arduino D2) |
| 3 | A1 | DATA_D0_3V3 (D42) |
| 4 | A2 | DATA_D1_3V3 (D43) |
| 5 | A3 | DATA_D2_3V3 (D44) |
| 6 | A4 | DATA_D3_3V3 (D45) |
| 7 | GND (note: pin 7 is GND on '4245A) | GND |
| 8 | A5 | DATA_D4_3V3 (D46) |
| 9 | A6 | DATA_D5_3V3 (D47) |
| 10 | A7 | DATA_D6_3V3 (D48) |
| 11 | A8 | DATA_D7_3V3 (D49) |
| 12 | GND | GND |
| 13 | B8 | DATA_D7_5V |
| 14 | B7 | DATA_D6_5V |
| 15 | B6 | DATA_D5_5V |
| 16 | B5 | DATA_D4_5V |
| 17 | GND (note: pin 17 is GND on '4245A) | GND |
| 18 | B4 | DATA_D3_5V |
| 19 | B3 | DATA_D2_5V |
| 20 | B2 | DATA_D1_5V |
| 21 | B1 | DATA_D0_5V |
| 22 | GND | GND |
| 23 | OE_N | GND |
| 24 | VCCB | 5V |

**Important:** Verify the exact pinout against the [SN74LVC4245A datasheet](https://www.ti.com/lit/ds/symlink/sn74lvc4245a.pdf) before routing. The pin mapping above is based on the TI datasheet but should be confirmed for the specific PWR (TSSOP-24) package variant.

### U6–U9: TXB0108PW (Unchanged)

- No changes to these ICs
- Keep R6–R9 (10K pull-ups on OE pins)
- Keep 3 decoupling caps each (VCCA + VCCB + extra)

### DIR Net (New)

- Add new net "DATA_DIR" from the Arduino Giga header pin D2 to U5 pin 2
- This is a direct 3.3V connection — does **not** go through any level shifter
- D2 must be routed from whichever header connector carries that pin to U5

### Removed Components

| Component | Was | Reason |
|-----------|-----|--------|
| R1 | 10K OE pull-up for U1 | OE tied to GND instead |
| R2 | 10K OE pull-up for U2 | OE tied to GND instead |
| R3 | 10K OE pull-up for U3 | OE tied to GND instead |
| R4 | 10K OE pull-up for U4 | OE tied to GND instead |
| R5 | 10K OE pull-up for U5 | OE tied to GND instead |
| C19–C27 | Decoupling caps | Reduced cap count (single-supply ICs need fewer caps) |

### Decoupling Cap Assignments

| IC | Caps | Rail(s) |
|----|------|---------|
| U1 (74LVC541) | C1 | VCC (3.3V) |
| U2 (74LVC541) | C2 | VCC (3.3V) |
| U3 (74LVC541) | C3 | VCC (3.3V) |
| U4 (74AHCT541) | C4 | VCC (5V) |
| U5 (SN74LVC4245A) | C5, C6 | VCCA (3.3V), VCCB (5V) |
| U6 (TXB0108) | C7, C8, C9 | VCCA (3.3V), VCCB (5V), extra |
| U7 (TXB0108) | C10, C11, C12 | VCCA (3.3V), VCCB (5V), extra |
| U8 (TXB0108) | C13, C14, C15 | VCCA (3.3V), VCCB (5V), extra |
| U9 (TXB0108) | C16, C17, C18 | VCCA (3.3V), VCCB (5V), extra |

## PCB Layout Changes

### Footprint Changes

- **U5**: Change from TSSOP-20 (4.4x6.5mm) to **TSSOP-24 (4.4x7.8mm)**
  - Same 0.65mm pitch, +1.3mm body length, 4 extra pins
  - U5 position may need slight adjustment to clear neighboring components
  - The wider body extends equally in both directions from center

### Trace Rerouting

- **U1–U3**: The 74LVC541 has a different pinout from TXB0108 (A/Y instead of A/B). All signal traces to U1–U3 need rerouting to match the new pin assignments.
- **U4**: Same pinout rerouting needed for 74AHCT541.
- **U5**: Complete reroute needed — different IC, different pinout, different footprint, plus new DIR trace.
- **DIR trace**: New trace from header pin (D2) to U5 pin 2. Route on whichever layer has space.
- **OE pins**: U1–U5 OE pins route to GND instead of pull-up resistors. Shorter traces.
- **Power traces**: U4 VCC changes from 3.3V to 5V. Ensure adequate trace width for the 5V power connection.

### Silkscreen Updates

- Change board label from "V0.1" to **"V0.2"**
- Update IC reference labels: U1–U3 = "74LVC541", U4 = "74AHCT541", U5 = "LVC4245A"
- Add "DIR" label near U5 pin 2 and the header connection
- Remove silkscreen for R1–R5 and C19–C27

### Design Rule Check

After all changes, run DRC to verify:
- No unconnected nets
- No clearance violations (especially around U5's wider footprint)
- All power nets correctly assigned
- DIR net connectivity from header to U5

## Gerber Regeneration

After completing all KiCad changes:
1. Run DRC and fix any errors
2. Export new Gerber files to `gerber/`
3. Export new drill files
4. Export updated schematic PDF to `schematic/`
5. Update component placement export to `cpl/`
6. Verify BOM matches schematic
