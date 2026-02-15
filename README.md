# RetroShield Level Shifter PCB — Arduino Giga R1 Shield

KiCad design files for a bidirectional 3.3V-to-5V level converter shield that sits between an [Arduino Giga R1 WiFi](https://store.arduino.cc/products/giga-r1-wifi) and a [RetroShield Z80](https://www.tindie.com/products/8bitforce/retroshield-for-arduino-mega-z80/). The shield translates all bus signals between the Giga's 3.3V logic and the RetroShield's 5V logic.

<p align="center">
  <img src="images/AlexJ_bz_ArduinoGigaShield_TOP.png" alt="3D render of the Arduino Giga R1 Level Shifter Shield — top view" width="600">
</p>

## Design Overview

**Version:** V0.2

V0.1 used nine identical TXB0108PW auto-direction-sensing level translators. In practice, the TXB0108's direction sensing failed for several Z80 bus signals: `IORQ_N` and `RD_N` were stuck HIGH, the data bus was invisible during IO writes, and `WR_N` was unreliable during IO cycles. The firmware worked around these failures with ~1,300 lines of shadow register tracking code.

V0.2 replaces 5 of the 9 TXB0108s with purpose-matched ICs that have explicit direction control, eliminating these failures entirely:

| Position | IC | Package | Role |
|----------|---|---------|------|
| U1, U2, U3 | **SN74LVC541** | TSSOP-20 | Address bus + control inputs (5V→3.3V) |
| U4 | **SN74AHCT541** | TSSOP-20 | Control outputs (3.3V→5V) |
| U5 | **SN74LVC4245A** | TSSOP-24 | Data bus (bidirectional, explicit DIR) |
| U6–U9 | TXB0108PW | TSSOP-20 | Pass-through GPIO (unchanged) |

**Board specifications:**
- **Dimensions:** 155mm x 90mm (matches Arduino Giga R1 footprint)
- **Layers:** 2-layer PCB
- **Signals translated:** 72 channels covering data bus, address bus, and control signals
- **DIR pin:** Arduino D2 → U5 pin 2 (controls data bus direction)

### Signal Groups

| Signal Group | Pins | Channels | Direction | IC |
|-------------|------|----------|-----------|-----|
| Address bus A0–A7 | D22–D29 | 8 | Z80 → Arduino (5V→3.3V) | U1 (74LVC541) |
| Address bus A8–A15 | D30–D37 | 8 | Z80 → Arduino (5V→3.3V) | U2 (74LVC541) |
| Control (MREQ, IORQ, RD, WR) | D39–D41, D53 | 4 | Z80 → Arduino (5V→3.3V) | U3 (74LVC541) |
| Control (CLK, RESET, INT, NMI) | D38, D50–D52 | 4 | Arduino → Z80 (3.3V→5V) | U4 (74AHCT541) |
| Data bus D0–D7 | D42–D49 | 8 | Bidirectional (DIR pin) | U5 (SN74LVC4245A) |
| Remaining GPIO | Various | 40 | Bidirectional (auto-sense) | U6–U9 (TXB0108) |

### IC Selection Rationale

- **74LVC541** (U1–U3): VCC=3.3V, inputs are 5V-tolerant. Unidirectional buffer — no direction sensing ambiguity. OE pins tied to GND (always enabled). No pull-up resistors needed.
- **74AHCT541** (U4): VCC=5V, TTL-compatible inputs accept 3.3V drive levels. Unidirectional buffer for Arduino→Z80 control signals. OE pins tied to GND.
- **SN74LVC4245A** (U5): VCCA=3.3V, VCCB=5V. Bidirectional transceiver with explicit DIR pin. DIR HIGH = A→B (Giga drives Z80 data bus), DIR LOW = B→A (Z80 drives Giga). OE tied to GND. DIR pin connected directly to Arduino D2 at 3.3V (does not go through a level shifter).
- **TXB0108PW** (U6–U9): Retained for pass-through GPIO where auto-direction sensing works fine. OE pins pulled HIGH via 10K resistors to 3.3V.

### DIR Pin

The SN74LVC4245A (U5) requires one GPIO from the Giga for direction control:

- **Arduino pin D2** (directly connected at 3.3V, no level shifter)
- **DIR HIGH** → A→B: Giga drives Z80 data bus (memory/IO writes to Z80)
- **DIR LOW** → B→A: Z80 drives Giga data bus (memory/IO reads from Z80)

This single GPIO wire replaces the entire shadow register architecture from v0.1.

## Repository Contents

```
kicad/                  KiCad 8 source files
  ├── AlexJ_bz_ArduinoGigaShield.kicad_pro    Project file
  ├── AlexJ_bz_ArduinoGigaShield.kicad_sch    Schematic
  ├── AlexJ_bz_ArduinoGigaShield.kicad_pcb    PCB layout
  └── AlexJ_bz_ArduinoGigaShield.kicad_prl    Project preferences

gerber/                 Production-ready Gerber files (v0.1 — regenerate from KiCad for v0.2)
  ├── *-F_Cu.gbr           Front copper
  ├── *-B_Cu.gbr           Back copper
  ├── *-F_Mask.gbr         Front solder mask
  ├── *-B_Mask.gbr         Back solder mask
  ├── *-F_Silkscreen.gbr   Front silkscreen
  ├── *-B_Silkscreen.gbr   Back silkscreen
  ├── *-F_Paste.gbr        Front paste (for stencil)
  ├── *-B_Paste.gbr        Back paste
  ├── *-Edge_Cuts.gbr      Board outline
  ├── *-PTH.drl            Plated through-hole drill
  ├── *-NPTH.drl           Non-plated through-hole drill
  └── *-job.gbrjob         Gerber job file

bom/                    Bill of Materials
  ├── AlexJ_bz_ArduinoGigaShield.csv     CSV format
  └── AlexJ_bz_ArduinoGigaShield.xlsx    Excel format (v0.1)

cpl/                    Component Placement
  └── AlexJ_bz_ArduinoGigaShield-all-pos.csv   Pick & place positions

schematic/              Schematic PDF (v0.1 — regenerate from KiCad for v0.2)
  └── AlexJ_bz_ArduinoGigaShield.pdf

images/                 3D renders
  ├── AlexJ_bz_ArduinoGigaShield_TOP.jpg       Top view
  ├── AlexJ_bz_ArduinoGigaShield_TOP.png       Top view (PNG)
  ├── AlexJ_bz_ArduinoGigaShield_BTM.jpg       Bottom view
  └── AlexJ_bz_ArduinoGigaShield.png           Perspective view

CHANGES-V0.2.md         Detailed design changes for KiCad implementation
```

## Bill of Materials

| Reference | Qty | Value | Part Number | Package |
|-----------|-----|-------|-------------|---------|
| U1, U2, U3 | 3 | SN74LVC541 | SN74LVC541PWR | TSSOP-20 |
| U4 | 1 | SN74AHCT541 | SN74AHCT541PWR | TSSOP-20 |
| U5 | 1 | SN74LVC4245A | SN74LVC4245APWR | TSSOP-24 |
| U6–U9 | 4 | TXB0108PW | TXB0108PWR | TSSOP-20 |
| C1–C18 | 18 | 0.1 uF | CC0603KRX7R9BB104 | 0603 |
| R6–R9 | 4 | 10K | RC0603FR-0710KL | 0603 |

**Cap breakdown:** U1–U3 get 1 cap each (VCC only, single supply), U4 gets 1 cap (VCC only), U5 gets 2 caps (VCCA + VCCB), U6–U9 get 3 caps each (VCCA + VCCB + extra) = 3 + 1 + 2 + 12 = 18 caps.

**Resistors:** R6–R9 are 10K pull-ups on OE pins for U6–U9 (TXB0108). R1–R5 from v0.1 are removed — the 74LVC541, 74AHCT541, and SN74LVC4245A have their OE pins tied directly to GND.

## Manufacturing

The Gerber files in `gerber/` are from v0.1. After the KiCad schematic and PCB are updated for v0.2, regenerate Gerbers before ordering.

Upload to any PCB fabrication service:

- [PCBWay](https://www.pcbway.com/)
- [JLCPCB](https://jlcpcb.com/)
- [OSH Park](https://oshpark.com/)

For SMD assembly, provide the BOM (`bom/`) and component placement file (`cpl/`) to the fab house.

### Recommended PCB Specs

| Parameter | Value |
|-----------|-------|
| Layers | 2 |
| Board thickness | 1.6 mm |
| Copper weight | 1 oz |
| Surface finish | HASL or ENIG |
| Solder mask | Red (or your preference) |
| Silkscreen | White |

## Assembly Notes

1. **SMD components first** — solder the ICs, decoupling capacitors, and pull-up resistors
2. **Pin headers last** — solder the through-hole pin headers (female on bottom for the Giga, male on top for the RetroShield)
3. **Power jumper** — a jumper wire connects the Giga's 3.3V output to the shield's VCCA rail
4. **DIR wire** — connect Arduino D2 to U5 pin 2 (DIR) if not routed on the PCB
5. **Orientation** — check silkscreen for "V0.2" and IC labels; U5 is the wider TSSOP-24 package

## Photos

<p align="center">
  <img src="images/AlexJ_bz_ArduinoGigaShield.png" alt="3D perspective render of the shield" width="500">
</p>

<p align="center">
  <img src="images/AlexJ_bz_ArduinoGigaShield_BTM.jpg" alt="3D render — bottom view showing routing" width="500">
</p>

## Related Projects

- [retroshield-z80-cpm-giga](https://github.com/ajokela/retroshield-z80-cpm-giga) — Arduino Giga R1 firmware for CP/M 2.2 on the RetroShield Z80
- [retroshield-sector-server](https://github.com/ajokela/retroshield-sector-server) — Rust TCP server for WiFi-based CP/M disk I/O

## Blog Posts

This project is documented in a three-part series on [tinycomputers.io](https://tinycomputers.io):

1. [My Experience Using Fiverr for Custom PCB Design: A $468 Arduino Giga Shield](https://tinycomputers.io/posts/fiverr-pcb-design-arduino-giga-shield/)
2. [Porting CP/M to the Arduino Giga R1: When Level Converters Fight Back](https://tinycomputers.io/posts/cpm-on-arduino-giga-r1-wifi/)
3. [Playing Zork on a Real Z80: From CP/M Boot to the Great Underground Empire](https://tinycomputers.io/posts/zork-on-retroshield-z80-arduino-giga/)

## License

This work is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/) (CC BY-SA 4.0). See [LICENSE](LICENSE).

You are free to share and adapt this design, including for commercial purposes, as long as you give appropriate credit and distribute derivative works under the same license.

## Author

Alex Jokela — [tinycomputers.io](https://tinycomputers.io)

PCB design by Elijah (ekeziah) via Fiverr. Manufacturing sponsored by [PCBWay](https://www.pcbway.com/).
