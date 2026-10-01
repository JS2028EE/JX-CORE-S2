# Component selection and BOM — Rev A draft

These selections were compared against the [2026-09-30 EasyEDA BOM export](exports/2026-09-30/BOM_Board1_Schematic1_2026-09-30.xlsx): 22 component instances in 14 grouped rows, each with the LCSC supplier number shown below. That verifies the exported assignments, **not** physical footprint geometry, LED polarity, or supplier stock. After any schematic edit, export a fresh BOM and compare it again before ordering. Quantities describe one board.

| Ref(s) | Qty | Part / value | LCSC | Why it is here | Verification |
| --- | ---: | --- | --- | --- | --- |
| U1 | 1 | ESP32-S2-MINI-2-N4 | [C3013906](https://www.lcsc.com/product-detail/C3013906.html) | Integrated ESP32-S2, 4 MB flash, 2.4 GHz Wi-Fi and PCB antenna; avoided laying out the RF chip/flash/crystal individually. | Confirm N4, not N4R2, and exact module footprint/antenna keepout. |
| USBC1 | 1 | TYPE-C-31-M-12, 16-pin USB-C receptacle | [C165948](https://www.lcsc.com/product-detail/C165948.html) | One physical port for 5 V power and native USB data. | Verify A/B pin mapping, shell tabs, footprint and board-edge fit. |
| U2 | 1 | AP2112K-3.3TRG1, SOT-23-5 LDO | [C51118](https://www.lcsc.com/product-detail/C51118.html) | Makes the module's 3.3 V rail from USB 5 V; nominal 600 mA regulator class. | Check VIN/GND/EN/NC/VOUT pin numbers, thermal budget and output at Wi-Fi peaks. |
| D1 | 1 | USBLC6-2SC6 USB ESD protector | [C7519](https://www.lcsc.com/product-detail/C7519.html) | Shunts ESD near the USB connector while protecting both data lines. | Confirm diode pin mapping, VBUS/GND and shortest practical path to connector/ground. |
| R1,R2 | 2 | 5.1 kΩ, 0603 | [C23186](https://www.lcsc.com/product-detail/C23186.html) | Separate CC1 and CC2 sink pull-downs for a basic 5 V USB-C sink. | One resistor per CC pin, each to GND; never short CC1 and CC2 together. |
| R3,R4 | 2 | 22 Ω, 0603 | [C23345](https://www.lcsc.com/product-detail/C23345.html) | Series footprints on D+ and D− per Espressif's initial USB guidance. | One in each line, close to the ESP32 module. |
| R5,R6 | 2 | 10 kΩ, 0603 | [C25804](https://www.lcsc.com/product-detail/C25804.html) | EN and IO0 pull-ups. | Confirm both go to 3V3 and the intended signal. |
| R7 | 1 | 330 Ω, 0603 | [C23138](https://www.lcsc.com/product-detail/C23138.html) | Chosen series resistor for the green rail indicator. | This is the current user choice; confirm R7's attached BOM/footprint. |
| C1,C2 | 2 | 10 µF, 16 V X5R, 0805 | [C1713](https://www.lcsc.com/product-detail/C1713.html) | Input bulk capacitance and output/3V3 reservoir for current transients. | Confirm effective capacitance with DC bias, values/footprints and placement. |
| C3 | 1 | 100 nF, 50 V X7R, 0603 | [C1591](https://www.lcsc.com/product-detail/C1591.html) | Local 3V3 high-frequency decoupling. | Place close to U1 power entry with short ground return. |
| C4 | 1 | 1 µF, 50 V X5R, 0603 | [C15849](https://www.lcsc.com/product-detail/C15849.html) | EN-to-GND power-up/reset RC with 10 kΩ. | Confirm this is on EN, not BOOT; account for capacitor DC-bias variation. |
| SW1,SW2 | 2 | TS-1187A-B-A-B tactile switch | [C318884](https://www.lcsc.com/product-detail/C318884.html) | BOOT and RESET buttons. | Check which two pads are already common inside each switch and the chosen footprint orientation. |
| H1–H4 | 4 | HX PZ2.54-1x10P ZZ, 2.54 mm through-hole header | [C42372502](https://www.lcsc.com/product-detail/C42372502.html) | Four accessible groups of ten signals. This matches the header model visible in the schematic screenshot. | Confirm each electrical pin has an actual connection and review physical pin-1 positions. |
| LED1 | 1 | Everlight 19-217/GHC-YR1S2/3T, green 0603 LED (label visible in the schematic) | [C72043](https://www.lcsc.com/product-detail/C72043.html) | Visible 3V3 power indicator. | Confirm this exact LCSC number is attached in EasyEDA, check footprint/cathode marking and brightness at 330 Ω. |

**LED selection note.** The 2026-09-30 schematic image labels `LED1` as Everlight 19-217/GHC-YR1S2/3T; LCSC maps that part to C72043. The manufacturer's catalogue gives 3.3 V typical forward voltage at 20 mA. At the board's 3.3 V supply with 330 Ω in series, the operating current and visibility cannot be inferred by plugging 3.3 V into the resistor formula because forward voltage changes with current. Test visibility or select a lower-forward-voltage green LED before ordering. The archived EasyEDA BOM confirms C72043 is assigned in this revision. Check its footprint and polarity on the PCB, then test visibility on a built board.

**Alternates and placement.** Earlier discussion also mentioned a 1 kΩ indicator resistor and a red LED. Those are **not** the current design. The 10 µF capacitors are **0805**, while most small passives are **0603**. This matters when assigning EasyEDA footprints. Purchase quantities may exceed board quantities because LCSC sells some parts in multiples.

Sources for electrical limits and layout are in [DESIGN_NOTES.md](DESIGN_NOTES.md). Date of this draft: 2026-09-30.
