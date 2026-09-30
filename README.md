# JX-CORE S2

**My custom ESP32-S2 development board, built one circuit at a time.**

I wanted to understand what actually goes into a microcontroller board: USB-C power and data, a stable 3.3 V rail, boot and reset control, protection, and a way to use the GPIO pins in my own projects. JX-CORE S2 is the board I am designing in EasyEDA around the **ESP32-S2-MINI-2-N4** module, with parts sourced through LCSC.

> **Status: schematic in progress (Rev A).** This repository documents the intended design and the parts selected so far. The EasyEDA source, ERC report, PCB layout, fabrication files, and physical test results have **not** been added yet. The board has not been verified or manufactured. Do not treat the tables here as proof that every connection is already present in the EasyEDA file.

## What it is supposed to do

- Run firmware on an ESP32-S2 single-core processor (up to 240 MHz) with 2.4 GHz Wi-Fi, 4 MB flash, and native full-speed USB.
- Take 5 V from a USB-C connector and regulate it to 3.3 V for the module.
- Let me enter USB download mode with a BOOT button and restart with a RESET button.
- Show that the 3.3 V rail is on with a **green** LED and **330 Ω** series resistor.
- Bring usable pins to **four 1×10, 2.54 mm headers** for experiments, sensors, and external driver circuits.

The selected **N4** module has **no PSRAM**. The ESP32-S2 has **Wi-Fi but no Bluetooth**. “High power” is a future system goal; this revision is a controller board and does not contain a high-current motor driver, switching output, battery charger, or USB Power Delivery circuit. GPIO pins are 3.3 V logic signals, not load power outputs.

## How the board is organized

| Block | Design |
| --- | --- |
| USB-C power | CC1 and CC2 each have their own 5.1 kΩ pull-down to GND; VBUS is `USB_5V`. |
| 3.3 V supply | AP2112K-3.3 regulator, input/output capacitors, local 100 nF decoupling. |
| USB data | Connector D−/D+ pairs combined by polarity, ESD protection near the connector, then a 22 Ω series resistor on each line to module IO19/IO20. |
| BOOT | IO0 held high with 10 kΩ and pulled to GND by the BOOT switch. |
| RESET | EN held high with 10 kΩ, 1 µF to GND for power-up delay, and pulled to GND by the RESET switch. |
| Indicator | `3V3 → 330 Ω → green LED anode → LED cathode → GND`. |
| Expansion | H1–H4 provide GPIO, GND, 3V3, the USB 5 V rail, and UART0 TX/RX. H3 pin 3 is IO45 and H4 pin 3 is input-only IO46; both need boot-strap care. BOOT/IO0 and RESET/EN stay on-board. |

The detailed [pin-by-pin header map](docs/PINOUT.md), [component selection](docs/BOM.md), and [calculations and layout notes](docs/DESIGN_NOTES.md) are separate so I can update them as I verify the schematic.

## What I have learned while making it

1. **A module pin number is not a GPIO number.** Module pin 45 is **EN**; the signal called **IO45** is on module pin 41. They are different connections.
2. **A pin name in EasyEDA is not a wire.** A header pin labeled `IO1` only reaches module IO1 when I connect it with a wire or matching net labels. I need to check the generated netlist, not just how the page looks.
3. **BOOT and RESET do different jobs.** IO0 changes the boot mode when sampled at reset. EN actually resets or enables the chip.
4. **Exposed does not mean unrestricted.** I changed the two header contacts from IO0 and EN to IO45 and IO46. IO46 works only as an input, and attached circuits on both pins can affect boot straps. I need to keep them low at the relevant reset/download time.
5. **The whole power path matters.** A regulator's “600 mA” rating is not a promise that my USB port, PCB traces, and SOT-23-5 package can deliver 600 mA continuously to everything attached.
6. **The LED resistor sets its current.** I chose to keep the green LED and 330 Ω resistor; the LED's actual forward-voltage curve and brightness still depend on the exact LED part.
7. **PCB placement matters as much as connectivity.** The USB traces need controlled routing and a return path, and the module antenna needs a keepout area.

## Rev A progress

- [x] Select module, USB connector, regulator, USB protection, buttons, passives, and four headers.
- [x] Draw the power, EN, IO0, USB, and indicator blocks in EasyEDA.
- [x] Connect the module GND pins in the working schematic, per the latest design update.
- [ ] Wire **every header contact electrically** to its matching module/net and verify via Design Manager/netlist.
- [ ] Assign and confirm the **actual green LED** LCSC part/footprint at D1 (a candidate is listed in the BOM).
- [ ] Run ERC and review every power, USB, BOOT, EN, and GND net; fix unintended open pins.
- [ ] Export and commit the EasyEDA source and a dated schematic PDF/PNG.
- [ ] Place and route the PCB with the antenna keepout, ground plane, short decoupling paths, and USB routing rules.
- [ ] Run PCB DRC; inspect footprint pin numbering and 3D orientation; export Gerbers/BOM/placement files.
- [ ] On a fabricated board, measure USB 5 V and 3.3 V, check startup current, test BOOT/RESET, flash firmware, test Wi-Fi and USB, and record results.

## Limitations and use

`USB_5V` at the header is the USB connector's VBUS net, **not** an independent high-current supply. The 3V3 header pins share the regulator with the ESP32 and the LED. Never apply 5 V to an ESP32 GPIO or the 3V3 rail. Do not feed a second power source into the exposed rails while USB is attached without first designing power-source isolation. Large loads need their own suitable supply and a driver or MOSFET stage with a common ground and appropriate protection. See [power calculations and restrictions](docs/DESIGN_NOTES.md).

## References

- [Espressif ESP32-S2-MINI-2 module datasheet](https://documentation.espressif.com/esp32-s2-mini-2_esp32-s2-mini-2u_datasheet_en.html)
- [Espressif ESP32-S2 schematic checklist](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s2/schematic-checklist.html)
- [Espressif ESP32-S2 PCB layout guidelines](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s2/pcb-layout-design.html)
- [Diodes Incorporated AP2112 datasheet](https://www.diodes.com/assets/Datasheets/AP2112.pdf)
- [Texas Instruments guide to USB Type-C](https://www.ti.com/lit/pdf/slyy228)
- Individual LCSC product links are in the [BOM](docs/BOM.md). Verify stock and substitutions when ordering; they change.

## License

My original repository documentation and any future original source files are available under the [MIT License](LICENSE). Component datasheets and third-party library symbols/footprints belong to their respective owners and are linked rather than relicensed here.
