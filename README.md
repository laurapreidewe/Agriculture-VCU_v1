# AgriVCU v1 — Agriculture Vehicle Control Unit

AgriVCU is an STM32F446RE-based vehicle control PCB derived from the Cultivator PCB. It provides relay control and interfaces for the main motor, pump motor, brakes, throttle signals, and peripheral modules.

**The editable KiCad design in this repository is the corrected version. Previously printed AgriVCU v1 boards have a VCAP-to-3V3 connection error and require the trace-cut repair below.** The Cultivator-to-AgriVCU v1 changes and this printed-board repair are separate revision history items.

## Printed AgriVCU v1: required VCAP repair

On the affected printed PCB, the STM32's VCAP pin was accidentally connected to the 3V3 vias beside it. To make the board work, **cut the trace between VCAP and the connected 3V3 via(s)**.

The corrected design identifies the relevant connections as:

| Connection | Corrected design |
| --- | --- |
| MCU | U2, STM32F446RETx |
| VCAP pin | U2 pin 30, `VCAP_1` |
| Dedicated VCAP net | `Net-(U2-VCAP_1)` |
| VCAP capacitor | C6, `1u`: pin 1 to VCAP; pin 2 to GND |
| 3V3 supply | `+3V3`, separate from the VCAP net |

1. Disconnect all power, including USB and the debugger, before working on the PCB.
2. Identify U2 pin 30, C6, and the trace branch connecting VCAP to the 3V3 via(s) on the actual printed board.
3. Cut that branch to isolate VCAP from 3V3. **Keep the U2-to-C6 connection intact.**
4. Inspect the cut and use a meter to confirm that the unintended copper short to 3V3 is removed, VCAP still reaches C6 pin 1, and C6 pin 2 remains connected to GND before powering the board again.

VCAP is the MCU's internal regulator output, not a 3V3 supply input. Use the current [PCB layout](Tractor%20VCU.kicad_pcb) and [MCU schematic](STM32F446RE%20minsys.kicad_sch) as the wiring reference. Updating the repository files does not repair an already printed board.

## Hardware overview

- **MCU:** STM32F446RET6, Arm Cortex-M4, LQFP-64.
- **PCB:** four copper layers. The revision list describes top and bottom signal/power routing, inner layer 1 ground, and inner layer 2 power.
- **Relay control:** five main-motor channels, five pump-motor channels, and two external relay channels.
- **Communication:** USB Mini-B, SWD, two CAN transceiver-module connectors, USART3, and an SPI1 SD-card-module connector.
- **I/O:** two brake-switch inputs, two ADC input connectors, DAC/throttle circuitry, and four MCU expansion headers.
- **Power:** connections labeled 12 V and 72 V input, with 12 V, 5 V, and 3V3 rail indicators and test points; external buck-converter connections are provided.

The supply labels do not specify allowable voltage ranges, current ratings, or relay load limits. Check the current schematics and fitted component/module ratings for the intended application. This repository contains hardware files and exports; firmware is not included.

## Repository contents

| File or folder | Purpose |
| --- | --- |
| [Tractor VCU.kicad_pro](Tractor%20VCU.kicad_pro) | KiCad project; open this first. |
| [Tractor VCU.kicad_sch](Tractor%20VCU.kicad_sch) | Top-level hierarchical schematic. |
| [STM32F446RE minsys.kicad_sch](STM32F446RE%20minsys.kicad_sch) | MCU, clock, reset, boot, and decoupling circuitry. |
| [power supply.kicad_sch](power%20supply.kicad_sch) | Power rails and buck-converter connections. |
| [VCU Relays.kicad_sch](VCU%20Relays.kicad_sch) | Relay and driver circuitry. |
| [VCU Connectors.kicad_sch](VCU%20Connectors.kicad_sch) | External interfaces and connectors. |
| [Tractor VCU.kicad_pcb](Tractor%20VCU.kicad_pcb) | Editable PCB layout. |
| [Tractor VCU.step](Tractor%20VCU.step) | Board assembly STEP model for mechanical CAD, approximately 54 MiB. |
| [Gerber.zip](Gerber.zip) | Supplied Gerber fabrication archive, including drill files, PCBWay BOM, placement CSV, and IPC netlist. Replaces the former `Gerber/` directory. |
| [Drill/](Drill/) | Separate plated/non-plated drill files and Gerber drill maps. |
| [Tractor VCU-odb.zip](Tractor%20VCU-odb.zip) | Supplied ODB++ manufacturing export. |
| [bom/ibom.html](bom/ibom.html) | Interactive BOM for assembly reference. |
| [bom/interactive BOM.zip](bom/interactive%20BOM.zip) | Packaged interactive BOM. |
| [sym-lib-table](sym-lib-table) | Project symbol-library table. |
| [docs/](docs/) | Original revision list and user manual. |

Manufacturing archives and BOMs are saved exports. Regenerate the required outputs from the current design when preparing a new fabrication release or after changing the PCB.

## Open the project and 3D model

1. Install **KiCad 10 or newer with support for these file versions**. Although the original project folder is named `KiCad/8.0`, the current files were saved by KiCad 10 and cannot be opened by KiCad 8 or 9.
2. Open `Tractor VCU.kicad_pro` and keep all four child schematic files beside the top-level schematic.
3. Open the schematic or PCB editor from the project manager.
4. For mechanical integration, download [Tractor VCU.step](Tractor%20VCU.step) and import it into a STEP-compatible CAD application.

Some component 3D references in the PCB use `${KICAD8_3DMODEL_DIR}` or paths from the original Windows machine. Configure those paths and supply the corresponding component models if rebuilding the KiCad 3D view. The exported board STEP file can be opened independently.

## Connector quick reference

Use the board's pin-1 markings and current schematics for connector orientation and wiring. The [user manual](docs/Tractor%20VCU%20User%20Manual.pdf) contains the detailed connector and test-point tables.

| Reference | Interface |
| --- | --- |
| J1 | USB Mini-B. |
| J2 | SWD / ST-LINK: 1 = 3V3, 2 = SWDIO/PA13, 3 = SWCLK/PA14, 4 = NRST, 5 = GND. |
| J3–J6 | Left, bottom, right, and top MCU expansion headers. |
| J7 / J8 | External relay 1 / 2: pin 1 = NO, pin 2 = COM. |
| J9 / J10 | Main-motor / pump-motor D-sub interfaces. |
| J11 / J12 | Brake-switch inputs: pin 1 = signal, pin 2 = GND. |
| J13 / J14 | CAN1 / CAN2 transceiver-module interfaces. |
| J15 / J16 | CAN1 / CAN2 test connectors. |
| J17 | SPI1 SD-card-module interface; see the corrected mapping below. |
| J18 | USART3: 1 = GND, 2 = TX/PC10, 3 = RX/PC11. |
| J19 / J20 | ADC1_IN12/PC2 and ADC1_IN13/PC3: 1 = 5 V, 2 = signal, 3 = GND. |
| J21 | Supply input: 1 = 12 V input, 2 = GND, 3 = GND, 4 = 72 V input. |

### J17 SPI mapping and manual correction

The manual's J17 table swaps the PB4/PB5 assignments for MOSI and MISO. The current schematic and PCB use:

| J17 pin | Signal | STM32 pin |
| --- | --- | --- |
| 1 | Not connected | — |
| 2 | SPI1_CS | PB6 |
| 3 | SPI1_SCK | PB3 |
| 4 | SPI1_MOSI | PB5 |
| 5 | SPI1_MISO | PB4 |
| 6 | 5 V | — |
| 7 | GND | — |

## Boot, debugging, and test points

- **JP1 / BOOT0:** pins 1–2 select high; pins 2–3 select low. Use BOOT0 low for normal flash boot and ST-LINK debugging.
- **JP2 / BOOT1 (PB2):** pins 1–2 select high; pins 2–3 select low. Keep its state defined rather than floating.
- With BOOT0 high, BOOT1 low selects system memory; BOOT1 high selects embedded SRAM.
- Connect an ST-LINK-compatible debugger through J2 using the pinout above.
- Rail checks: **TP1 = 12 V, TP2 = 5 V, TP3 = 3V3**. TP19–TP23 provide nearby ground test points.
- Rail LEDs: **LED1 = USB VBUS, LED14 = 5 V, LED15 = 3V3, LED16 = 12 V**. The manual also identifies reset and power switches, relay LEDs, and fuse locations.

## Cultivator PCB → AgriVCU v1 changes

The supplied [Revision List.docx](docs/Revision%20List.docx) records the changes made from the Cultivator PCB to AgriVCU v1:

| Area | Change |
| --- | --- |
| Power routing | Move the 12 V route to an outer layer. |
| Ground layout | Separate high-current and low-current ground-plane areas. |
| MCU decoupling | Improve capacitor placement around the STM32. |
| MCU component packages | Change STM32-circuit resistor and capacitor footprints from 0603 to 0402. |
| Expansion | Add MCU extension headers. |
| Reset | Correct the reset circuit with a pull-up, reset switch, and capacitor. |
| USB | Revise the Mini-B USB circuitry. |
| Reverse-voltage protection | Add protection in the `5_BUCK` → `+5V` path. |
| Test access | Add ground points near the signal test points. |
| Buck connections | Change the buck-converter input/output connections and labeling. |

The revision list also explains the four-layer arrangement and KiCad's DNP (Do Not Place) markings. It does not describe the printed-board VCAP fault; that repair is documented separately above. Schematic title-block revision numbers identify individual sheet revisions and should not be confused with the AgriVCU v1 board name.

## Reference documents

- [Tractor VCU User Manual.pdf](docs/Tractor%20VCU%20User%20Manual.pdf): original hardware usage reference, connector tables, boot settings, indicators, and test points. Apply the J17 correction above and use the current KiCad files for wiring.
- [Revision List.docx](docs/Revision%20List.docx): original Cultivator-to-AgriVCU v1 comparison, with before/after diagrams.

The printed-board VCAP repair is documented from the board owner's hardware clarification; it is absent from both original documents. The current KiCad sources take precedence over older reference diagrams and saved exports when checking the corrected design.
