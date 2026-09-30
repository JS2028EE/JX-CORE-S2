# Pinout and electrical connections — Rev A

**Read the labels carefully:** `IO45` is a GPIO signal on module pad **41**. Module pad **45** is `EN`, the reset/enable input. The numbers in the first columns below are **header contact numbers**; the last column is the **ESP32-S2-MINI-2 module pad number**, where applicable. Header pin ordering must be checked against the EasyEDA footprint's pin-1 marker before fabrication.

## Four 10-pin headers

| Header | Pin | Signal/net | Module pad | Note |
| --- | ---: | --- | ---: | --- |
| H1 | 1 | GND | — | Ground |
| H1 | 2 | 3V3 | 3 | Regulated rail |
| H1 | 3 | IO1 | 5 | GPIO / ADC1 capable |
| H1 | 4 | IO2 | 6 | GPIO / ADC1 capable |
| H1 | 5 | IO3 | 7 | GPIO / ADC1 capable |
| H1 | 6 | IO4 | 8 | GPIO / ADC1 capable |
| H1 | 7 | IO5 | 9 | GPIO / ADC1 capable |
| H1 | 8 | IO6 | 10 | GPIO / ADC1 capable |
| H1 | 9 | IO7 | 11 | GPIO / ADC1 capable |
| H1 | 10 | IO8 | 12 | GPIO / ADC1 capable |
| H2 | 1 | IO9 | 13 | GPIO / ADC1 capable |
| H2 | 2 | IO10 | 14 | GPIO / ADC1 capable |
| H2 | 3 | IO11 | 15 | GPIO / ADC2 capable |
| H2 | 4 | IO12 | 16 | GPIO / ADC2 capable |
| H2 | 5 | IO13 | 17 | GPIO / ADC2 capable |
| H2 | 6 | IO14 | 18 | GPIO / ADC2 capable |
| H2 | 7 | IO15 | 19 | GPIO / ADC2 capable |
| H2 | 8 | IO16 | 20 | GPIO / ADC2 capable |
| H2 | 9 | IO17 | 21 | GPIO / ADC2 / DAC1 capable |
| H2 | 10 | IO18 | 22 | GPIO / ADC2 / DAC2 capable |
| H3 | 1 | GND | — | Ground |
| H3 | 2 | 3V3 | 3 | Regulated rail |
| H3 | 3 | IO45 | 41 | Strapping pin; keep low at reset for 3.3 V flash supply |
| H3 | 4 | IO21 | 25 | GPIO |
| H3 | 5 | IO26 | 26 | Available on selected **N4**, unavailable on N4R2 with PSRAM |
| H3 | 6 | IO33 | 28 | GPIO |
| H3 | 7 | IO34 | 29 | GPIO |
| H3 | 8 | IO35 | 31 | GPIO |
| H3 | 9 | IO36 | 32 | GPIO |
| H3 | 10 | IO37 | 33 | GPIO |
| H4 | 1 | GND | — | Ground |
| H4 | 2 | USB_5V | — | Connector VBUS; USB-powered only |
| H4 | 3 | IO46 | 44 | Input only; strapping pin, keep low for download mode |
| H4 | 4 | IO38 | 34 | GPIO |
| H4 | 5 | IO39 | 35 | GPIO / JTAG MTCK default |
| H4 | 6 | IO40 | 36 | GPIO / JTAG MTDO default |
| H4 | 7 | IO41 | 37 | GPIO / JTAG MTDI default |
| H4 | 8 | IO42 | 38 | GPIO / JTAG MTMS default |
| H4 | 9 | TXD0 / IO43 | 39 | UART0 TX, 3.3 V logic |
| H4 | 10 | RXD0 / IO44 | 40 | UART0 RX, 3.3 V logic |

The module's **IO19 (pad 23)** and **IO20 (pad 24)** are dedicated here to USB D− and D+. IO0 and EN are connected to their on-board BOOT and RESET circuits and are **not exposed on these headers**. IO45 and IO46 are exposed instead, with the strapping restrictions below. Module pad 27 is `NC`. Refer to the [module datasheet](https://documentation.espressif.com/esp32-s2-mini-2_esp32-s2-mini-2u_datasheet_en.html) for alternate pin functions and boot straps.

## Schematic nets, step by step

| Net / path | Required connection |
| --- | --- |
| GND | USB connector ground and shield, AP2112K GND, ESD GND, capacitor returns, BOOT/RESET switch ground sides, LED cathode, header GNDs, **all** module GND pads (1, 2, 30, 42, 43, 46–65). |
| USB_5V | USB-C VBUS contacts → AP2112K VIN and EN, input capacitor, ESD device VBUS/reference connection (per exact symbol), H4 pin 2. **Never connect it directly to module 3V3/GPIO.** |
| 3V3 | AP2112K VOUT → module pad 3, output/local decoupling capacitors, EN/IO0 pull-ups, indicator resistor, H1 pin 2, H3 pin 2. |
| CC1 / CC2 | Each USB-C CC contact → its **own** 5.1 kΩ resistor → GND. |
| USB_D− | Connector A7/B7 combined → ESD path → 22 Ω series resistor → module IO19/pad 23. |
| USB_D+ | Connector A6/B6 combined → ESD path → 22 Ω series resistor → module IO20/pad 24. |
| IO0 / BOOT | Module pad 4 → 10 kΩ pull-up to 3V3 and momentary button to GND. No header connection. |
| EN / RESET | Module pad 45 → 10 kΩ pull-up to 3V3, 1 µF capacitor to GND, and momentary button to GND. No header connection. |
| IO45 | Module pad 41 → H3 pin 3. Strap-sensitive; avoid an external HIGH at reset. |
| IO46 | Module pad 44 → H4 pin 3. Input only; avoid an external HIGH during download-mode entry. |
| PWR_LED | 3V3 → 330 Ω resistor → green LED anode → green LED cathode → GND. |

The module has its own internal flash and antenna. The exposed signals are 3.3 V logic; attached peripherals need their own current and voltage checks. IO45 and IO46 have internal weak pull-downs for their default strap values. An external pull-up or driven HIGH on H3 pin 3 (IO45) at reset can select 1.8 V for the flash supply and prevent booting. H4 pin 3 (IO46) must be LOW when entering download mode with IO0 LOW; it cannot drive an output at all. Label these contacts `IO45 STRAP` and `IO46 IN/STRAP` on the PCB, and test boot and flashing with attached peripherals. The BOOT button still pulls IO0 low at reset to select the download path.

## Suggested firmware interface map

These names describe **capabilities and recommended firmware assignments**, not extra fixed wiring inside the PCB. Most digital peripherals can be routed to other suitable GPIOs through the ESP32-S2 GPIO matrix.

| Function | GPIO | Header contact | Note |
| --- | --- | --- | --- |
| I²C SDA | IO7 | H1 pin 9 | Suggested shared data line |
| I²C SCL | IO8 | H1 pin 10 | Suggested shared clock line |
| SPI2 CS0 | IO10 | H2 pin 2 | Direct IO_MUX chip select for first device |
| SPI2 MOSI | IO11 | H2 pin 3 | Direct IO_MUX data out |
| SPI2 SCLK | IO12 | H2 pin 4 | Direct IO_MUX clock |
| SPI2 MISO | IO13 | H2 pin 5 | Direct IO_MUX data in |
| UART0 TX | IO43 / TXD0 | H4 pin 9 | Board transmit → peripheral receive |
| UART0 RX | IO44 / RXD0 | H4 pin 10 | Board receive ← peripheral transmit |

Every exposed IO1–IO18, IO21, IO26, and IO33–IO45 can be ordinary digital input or output when not assigned to another function. IO46 on H4 pin 3 is **input only**. IO0 is on the BOOT button circuit and is not header-accessible. IO19 and IO20 are used for on-board USB D−/D+ and should stay reserved for USB in this design. IO26 is available only with the selected N4 module; the N4R2 variant uses it for PSRAM.

For analog input, **ADC1** is IO1–IO10 (H1 pins 3–10 and H2 pins 1–2); **ADC2** is IO11–IO18 on H2 pins 3–10. IO19/IO20 also have ADC2 functions on the chip but are already occupied by USB. Prefer ADC1 for sampling while Wi-Fi is active because ADC2 is shared with Wi-Fi and readings may be disrupted. Analog output is available from the two 8-bit DAC channels on **IO17/H2 pin 9** and **IO18/H2 pin 10**. An ADC input is still subject to the configured attenuation and voltage limits; do not assume its full-scale measurement range is exactly 0–3.3 V.

I²C needs SDA and SCL pull-ups to **3V3**. A typical starting point is 4.7 kΩ on each line; check whether a connected sensor module already supplies them before adding more. The proposed SPI and I²C assignments use different pins, so both buses can be used together. Firmware must explicitly select these GPIO numbers.

## EasyEDA connection check

The red text printed inside a header symbol is **its pin name**, not a net label. Add an electrical wire from each header pin endpoint, then either connect it directly to the intended signal or put the **same** net label on short wires at both ends. Example: H1 pin 3 wire labeled `IO1` and U1 pad 5 wire labeled `IO1`. Ground and power flags must actually land on a wire or pin. Use the Design Manager to see which pins belong to each net, refresh ERC after edits, and inspect the exported netlist before PCB conversion. The [EasyEDA wiring guide](https://docs.easyeda.com/en/Schematic/Wiring-Tools/) describes net labels and their behavior.
