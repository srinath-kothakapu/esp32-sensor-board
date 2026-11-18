# ESP32 Sensor Board (BME280)

A small 2-layer PCB with an ESP32 module and a BME280 temperature / humidity / pressure sensor, powered from 5 V. The repository contains the KiCad 7 schematic and PCB layout, a bill of materials and the design-check results.


## Read this first (honest status)

- **Made with AI assistance.** This board was created with Claude (an AI assistant): it generated the KiCad schematic and PCB files from a part and net list, and ran the checks listed below. I am using the project to learn PCB design.
- **Not manufactured, not tested.** The board has never been built. Everything here is verified only by software checks (netlist comparison and KiCad design rule check).
- **Layout by a small script.** The traces were placed by a simple grid router that Claude wrote, not by hand. A real design would be routed and reviewed by hand in KiCad.

## What the board does

| Block | Parts | Job |
|---|---|---|
| Power | J1, C1, U2 (AMS1117-3.3), C2, C3 | 5 V in, 3.3 V out for the ESP32 and the sensor |
| Microcontroller | U1 (ESP32-WROOM-32), R1, C4, SW1, SW2 | Wi-Fi/Bluetooth MCU with reset (EN) and boot (GPIO0) buttons |
| Sensor | U3 (BME280), C5, R2, R3 | Temperature, humidity and pressure over I2C |
| Indicator | R4, D1 | Green LED on GPIO2 |
| Programming | J2 (GND / TXD / RXD) | Flash the ESP32 with a 3.3 V USB-UART adapter |

## Pin map

| Signal | ESP32 pin | Connected to |
|---|---|---|
| SDA | GPIO21 | BME280 SDI, R2 pull-up to 3.3 V |
| SCL | GPIO22 | BME280 SCK, R3 pull-up to 3.3 V |
| LED | GPIO2 | R4 (1 kOhm) then LED D1 to GND |
| BOOT | GPIO0 | SW2 to GND |
| EN | EN | R1 10 kOhm pull-up, C4 1 uF to GND, SW1 to GND |
| UART TX / RX | GPIO1 / GPIO3 | J2 header |

The BME280 runs in I2C mode: CSB is tied to 3.3 V and SDO to GND, which gives the I2C address 0x76.

## Design decisions

- **AMS1117-3.3 regulator:** simple and cheap for a 5 V input. It is a linear regulator, so it wastes the voltage difference as heat. A buck converter would be better for battery use.
- **EN circuit:** 10 kOhm pull-up with a 1 uF capacitor gives a clean power-up reset, and SW1 pulls EN low to reset.
- **Boot button on GPIO0:** holding it low at reset puts the ESP32 into flash mode.
- **I2C pull-ups:** 4.7 kOhm on SDA and SCL, one pair on the board.
- **Antenna:** the module sits at the top edge with its antenna end over the board edge, and a copper keepout is placed under and around the antenna area.
- **Ground:** a copper pour on both layers, tied together with stitching vias.
- **Unused ESP32 pins:** marked as not connected in the schematic.

## Design checks done

- **Netlist match:** the netlist exported from the schematic and the net of every pad on the PCB are identical (52 of 52 connections).
- **KiCad DRC (0.2 mm clearance):** 0 errors, 19 warnings (see `docs/drc_report.txt`). Two warnings come from the ESP32 outline overhanging the board edge on purpose, 17 from the footprint library not being configured.
- **Not done:** the schematic electrical rules check (ERC) was not run in the KiCad GUI, no 3D check, no manufacturer DFM check.

## Known limitations

- The 22 uF output capacitor of the AMS1117 should be a tantalum or low-ESR type. A plain ceramic can make this regulator unstable.
- The BME280 is an LGA package (2.5 x 2.5 mm), which is hard to hand-solder. A breakout module or hot-air soldering is needed.
- There is no USB connector and no automatic reset circuit, so flashing needs the boot button and an external USB-UART adapter.
- No reverse-polarity or over-current protection on the 5 V input.
- The ground pour is not filled in the saved PCB file. Open it in KiCad and press **B** to refill the zones.

## Files

```
hardware/
  esp32-sensor-board.kicad_sch   schematic (KiCad 7)
  esp32-sensor-board.kicad_pcb   2-layer PCB layout, 40 x 60 mm
  esp32-sensor-board.kicad_pro   KiCad project file
docs/
  bom.csv                        bill of materials
  drc_report.txt                 design check results
```

## Open it

Install KiCad 7 or newer, open `hardware/esp32-sensor-board.kicad_pro`, then open the schematic or the PCB editor. In the PCB editor press **B** to refill the ground zones.

## Next steps

- Route the board by hand and compare it with the scripted layout.
- Run ERC and a manufacturer DFM check, then order a prototype.
- Replace the linear regulator with a buck converter and add USB-C with auto-reset.
