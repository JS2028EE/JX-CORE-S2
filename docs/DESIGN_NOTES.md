# Design notes, math, limits, and verification

This is a **design record**, not a lab report. Numbers below are engineering estimates based on nominal values; hardware measurements and the green LED's real current–voltage curve will decide the actual results. See the [BOM](BOM.md) and [pinout](PINOUT.md) for part links and exact connections.

## Why these choices

- **ESP32-S2-MINI-2-N4**: a 240 MHz single-core, 2.4 GHz Wi-Fi module with USB, 4 MB built-in flash and on-board antenna. Using the module keeps the RF network, crystal, and flash implementation inside a manufacturer-designed assembly. This N4 variant has no PSRAM or Bluetooth. Availability motivated the move away from the earlier ESP32-S2 option; supplier stock still must be checked before ordering.
- **USB-C with two 5.1 kΩ CC pull-downs**: straightforward 5 V sink port. The two resistors let a Type-C source detect this device in either plug orientation. There is no USB Power Delivery negotiation.
- **AP2112K-3.3**: a compact 3.3 V regulator with a nominal 600 mA rating, suitable as the core rail when its temperature and total load are checked. Input/output ceramic capacitors and the local 100 nF part support supply stability and RF current steps.
- **USBLC6-2SC6 and two 22 Ω series resistors**: ESD protection at the connector and initially recommended USB series elements near the module. The actual routing, symbol pin assignment, and footprint must be checked.
- **BOOT and RESET buttons**: IO0 sets a boot strap when the chip resets; EN is the chip enable/reset. A 10 kΩ + 1 µF EN network follows Espressif's suggested starting point for power-up timing.
- **Green LED with 330 Ω**: a visible power indicator chosen by the project owner. The resistor prevents unbounded LED current. This is a rail indicator, not a firmware-controlled status LED.
- **Four 10-pin headers**: access to sensors, UART, control pins, 3.3 V and GND. H3 pin 3 is IO45 and H4 pin 3 is input-only IO46; both are boot straps. BOOT/IO0 and RESET/EN stay on their local button circuits. Header names alone do not establish connectivity; the netlist must prove it.

## Calculations

### EN power-up network

Nominal RC time constant:

`τ = R × C = 10,000 Ω × 1 µF = 10 ms`.

This is a characteristic rise time, **not** a guaranteed 10 ms reset interval. Tolerances, capacitor bias, the rail ramp, and the EN threshold affect actual release timing. Espressif recommends this 10 kΩ / 1 µF starting network and advises checking the actual supply behavior. Pressing RESET pulls EN low; the pull-up's steady resistor current while pressed is approximately `3.3 V / 10 kΩ = 0.33 mA`.

### BOOT pull-up

If BOOT is held down, the 10 kΩ resistor draws approximately `3.3 V / 10 kΩ = 0.33 mA`. The GPIO0 strap is sampled around reset; pressing BOOT alone while already running does not reset the chip. For manual USB download mode: **hold BOOT, tap RESET, release RESET, then release BOOT**. Firmware/tool behavior may offer other entry methods after the board is working.

### Green LED and resistor

`I_LED ≈ (3.3 V − V_F) / 330 Ω` when the LED conducts. For an **illustrative** lower-forward-voltage green LED with `V_F = 2.1 V` at its actual operating current, `I ≈ 3.64 mA` and the resistor dissipates `I²R ≈ 4.36 mW`. The schematic instead labels `LED1` as Everlight **19-217/GHC-YR1S2/3T**, LCSC **C72043**. Everlight lists 3.3 V typical forward voltage at **20 mA**, so that data point cannot predict the LED's current or visibility on a 3.3 V rail with 330 Ω. Expect possible dimness and test it or choose a lower-forward-voltage green part. The chosen resistor C23138 is rated 100 mW, well above the illustrative dissipation.

### Regulator heat and rail current

An LDO burns the voltage difference as heat: `P_LDO ≈ (5.0 V − 3.3 V) × I_3V3 = 1.7 V × I_3V3`, ignoring its small ground current.

| Total 3V3 current | Estimated regulator heat |
| ---: | ---: |
| 100 mA | 0.17 W |
| 200 mA | 0.34 W |
| 300 mA | 0.51 W |
| 500 mA | 0.85 W |

`I_3V3` includes **the ESP32, LED, and everything powered from either 3V3 header**. The AP2112K's 600 mA headline is an IC current capability, not a guaranteed board output allowance. Verify junction temperature using the actual PCB copper/thermal resistance, USB source/cable limit, and observed Wi-Fi load. For larger loads, use an appropriately designed separate supply; motors and solenoids also need driver and transient protection.

### USB-C CC and data

CC1 and CC2 each get a separate 5.1 kΩ pull-down (`Rd`) to GND for a basic sink. The source decides what current is advertised; this board does not negotiate USB PD voltages. D− routes to module IO19 and D+ routes to IO20. The 22 Ω components are **series resistors**, one per line; they do not go to ground. Espressif lists 22 or 33 Ω as initial values to reserve near the chip.

## Restrictions and limitations

1. **3.3 V logic only.** ESP32-S2 GPIOs are not 5 V tolerant. Use level shifting or compatible 3.3 V peripherals where needed; do not attach 5 V logic directly.
2. **USB_5V is the same VBUS net as the connector.** An external supply placed on that header can back-feed a computer or USB source. The present design has no reverse-power, fuse, load switch, or source-selection circuit documented.
3. **No high-power outputs.** The headers do not make GPIO pins motor drivers. A MOSFET/driver, external supply, shared ground, and inductive-load protection belong in a separate stage.
4. **No battery charging or regulation from a Li-ion cell.** This revision expects USB 5 V. A battery path requires a charger, protection, and power-path design before connection.
5. **No Bluetooth, PSRAM, 5 GHz Wi-Fi, or USB Power Delivery.** The ESP32-S2-MINI-2-N4 supplies 2.4 GHz Wi-Fi and 4 MB flash. USB is full-speed USB, not high-speed 480 Mb/s.
6. **Boot straps matter.** IO0 stays on the BOOT circuit, not the header. H3 pin 3 exposes IO45: an external HIGH at reset can select 1.8 V flash supply and stop the N4 board from booting. H4 pin 3 exposes IO46: it is input only and must be LOW when entering download mode with IO0 LOW. Both straps default LOW through weak internal pull-downs; external peripherals must not override them at reset. IO26 is available on N4 but not the N4R2 PSRAM variant.
7. **Header rails have a shared budget.** The connector's per-contact current rating does not increase the regulator or USB source capacity. Do not assume any maximum load without thermal and supply testing.

## PCB layout targets

- Place the module's **PCB antenna at the board edge**, ideally extending beyond the base board, with the manufacturer's no-copper/no-component keepout. If that placement is impossible, follow Espressif's illustrated clearance and validate RF performance. Keep USB, UART, headers, metal, and housing away from the antenna.
- Tie **all module ground pads** (1, 2, 30, 42, 43, and 46–65) to the GND net and provide a continuous ground return with appropriate vias/ground copper. Review the exact module land pattern and paste strategy for the exposed pads.
- Put the regulator input/output capacitors close to its pins and the module decoupler close to its 3V3/GND connection. Route 3V3 with enough width and minimal voltage drop.
- Put the USB ESD device near the receptacle with a short GND path; place USB series resistors near the module. Route D+/D− as a short, matched pair over continuous ground. Espressif's target is **90 Ω differential ±10%** based on the actual board stackup; do not guess a trace width from another PCB.
- Verify connector mechanical retention/shell pads and pin-1 markers for all four headers and both switches. Run DRC after routing and compare the PCB nets against the schematic.

Espressif's general chip guidelines include advice for bare IC designs as well as module designs. For this project, the **MINI-2 module datasheet and its recommended land pattern** govern module footprint and antenna placement. A two-layer board needs special attention to continuous ground under digital/USB regions and the antenna exclusion region.

## Verification plan before calling it a working board

1. The [2026-09-30 exported netlist review](NETLIST_REVIEW_2026-09-30.md) confirms the header map, USB data, 3V3, IO0, and EN, but finds USB connector GND, U2 GND, and C1 return isolated on `$1N3`. Join them to `GND` in EasyEDA, rerun DRC, and export a new netlist to prove the fix. The archived DRC log reports zero errors and one floating-pin warning for U1.27 (NC) and the USB-C SBU pins A8/B8; these may be marked No Connect only after confirming they are intentionally unused.
2. The 2026-09-30 BOM export confirms the LCSC codes for all 22 component instances, including green LED1/C72043 and ESD D1/C7519. Still inspect physical pad geometry/polarity, especially U1's footprint titled `WIFI-SMD_ESP32-MINI-1-N4` against the selected MINI-2 module datasheet, C1/C2 0805, USB-C receptacle, switches, LED cathode, and antenna keepout. Correct the schematic title block from ESP32-S3 to JX-CORE S2 in the next export.
3. Check PCB layout visually, calculate USB trace geometry for the board stackup, then run DRC and inspect the generated Gerbers before ordering.
4. On first power-up, use a current-limited source and measure USB_5V and 3V3 with no external peripherals. Check heating, supply drop, and switch behavior.
5. Flash a small GPIO/USB test program, then test Wi-Fi, USB enumeration in both Type-C plug orientations, RESET/BOOT, LED visibility, and all mapped header pins. Verify boot and download mode with any intended circuits on IO45/IO46; test IO46 as an input only. Record the date, photos, measurements, failures, and fixes in this repository.

## Primary references

- [Espressif MINI-2 datasheet: variant table, module pads, boot straps, land pattern](https://documentation.espressif.com/esp32-s2-mini-2_esp32-s2-mini-2u_datasheet_en.html)
- [Espressif ESP32-S2 schematic checklist: power, EN RC, USB](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s2/schematic-checklist.html)
- [Espressif ESP32-S2 PCB layout guidelines: module/antenna and USB routing](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s2/pcb-layout-design.html)
- [Diodes Incorporated AP2112 family datasheet](https://www.diodes.com/assets/Datasheets/AP2112.pdf)
- [Texas Instruments USB Type-C guide: 5.1 kΩ sink CC resistors](https://www.ti.com/lit/pdf/slyy228)
- [EasyEDA Standard wiring guide: wires and net labels](https://docs.easyeda.com/en/Schematic/Wiring-Tools/)
