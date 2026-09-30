# Exported netlist and BOM review — 2026-09-30

Inputs: EasyEDA Pro `Netlist_Schematic1_2026-09-30.enet` and `BOM_Board1_Schematic1_2026-09-30.xlsx`, exported from `Board1 : Schematic1`. This is a review of those exports, not a review of a corrected schematic or finished PCB.

## Required schematic fix before PCB conversion

The connector's **actual USB ground contacts** `USBC1.A1B12` and `USBC1.B1A12`, regulator `U2.2` (GND), and input capacitor `C1.2` share the unnamed net **`$1N3`**. The ESP32 module grounds, USB shield `USBC1.1`–`.4` (EH), ESD ground `D1.2`, output capacitor `C2.2`, and header grounds are on a *different* net, **`GND`**. The regulator input circuit therefore has no intended return connection to the board ground. The fact that the shield is on `GND` does not establish that the connector's GND contacts are on `GND`.

**Fix:** In EasyEDA Pro, attach a `GND` net flag or a real wire to the existing wire joining `USBC1.A1B12`, `USBC1.B1A12`, `U2.2`, and `C1.2`. It must make electrical contact with that wire, not merely overlap visually. Refresh schematic DRC and export a *new* `.enet`. Confirm each of those four pins now reports `net: GND`, and that `$1N3` disappears. Then update/convert the PCB from the corrected schematic. Do not order from the current export.

## Connections that passed the exported-net check

- USB VBUS contacts, regulator VIN/EN, input capacitor, ESD VBUS, and H4.2 are on `USB_5V`. Regulator VOUT, module 3V3, output/local capacitors, pull-ups, LED resistor, and H1.2/H3.2 are on `3V3`.
- CC1 and CC2 are separate, each reaching GND through its own 5.1 kΩ resistor.
- USB D+ uses connector A6/B6 → ESD D1.3/D1.4 → R3 22 Ω → module IO20/pad 24. D− uses connector A7/B7 → ESD D1.1/D1.6 → R4 22 Ω → module IO19/pad 23. ESD D1.2 is on GND and D1.5 on USB_5V.
- BOOT: IO0/pad 4 → R6 10 kΩ to 3V3, SW2 to GND. RESET: EN/pad 45 → R5 10 kΩ to 3V3, C4 1 µF to GND, SW1 to GND.
- All listed module GND pads 1, 2, 30, 42, 43, and 46–65 are on `GND`. NC pad 27 remains unconnected.
- H1–H4 each have exactly the [documented](PINOUT.md) ten nets. H3.3 is IO45/pad 41; H4.3 is input-only IO46/pad 44. IO0 and EN are absent from the headers.
- The exported BOM totals 22 components, and every listed row has an LCSC supplier part number matching the [design BOM](BOM.md), including D1/C7519 and LED1/C72043.

## Physical check before fabrication

The netlist assigns U1 the manufacturer part **ESP32-S2-MINI-2-N4**, but its footprint is titled **`WIFI-SMD_ESP32-MINI-1-N4`**. A name mismatch is not proof of a wrong pad pattern. Compare its physical pads, pitch, numbered locations, body outline, and antenna keepout against Espressif's [MINI-2 recommended land pattern](https://documentation.espressif.com/esp32-s2-mini-2_esp32-s2-mini-2u_datasheet_en.html), Figure 10-1. Do not approve fabrication based on the footprint title alone. Review the USB-C connector footprint and switch common pads in the same way.

After the ground fix, repeat the netlist check, then inspect the routed PCB with DRC and Gerber/3D review. The schematic DRC previously reported zero errors but did not catch the two separate ground nets.
