# Screw Jack Monitoring: Sensor Review and LabVIEW Integration Assessment

## Summary

All of the listed sensors can feed one LabVIEW screen, but not as listed:

- Two items (Fluke 62 MAX, Extech MA63) have no data output at all.
- One item (the "FM" VTV122) is not a handheld meter. It is an ifm transmitter.
- The MCC DAQs accept voltage (plus thermocouples). The 4–20 mA and Pt1000 signals need extra hardware.

**Verification status.** The ifm, Celesco and Dwyer datasheets were read directly. The Fluke and Extech findings come from published summaries. The MCC datasheets could not be read (downloads were blocked), so the MCC figures below are from memory and must be checked against the MCC links at the end.

## Per-device findings

| Device | PC-readable? | What you need |
|---|---|---|
| **Fluke 62 MAX** (IR thermometer) | **No.** Handheld, no USB, Bluetooth or logging. | Keep for spot checks. Log temperature with thermocouples (K or T type) on gearbox and motor housings into the USB-2408. |
| **Extech MA63** (clamp meter) | **No.** Display only. | Keep as a cross-check. For logging, use true-RMS current transducers with a 4–20 mA or 0–10 V output, one per motor. |
| **Celesco SG1-80-3** (string pot) | **Yes.** Passive 10 kΩ voltage divider, output about 94 ±4 % of the excitation voltage, M12 connector. | Clean 5–10 V excitation (1 mA at 10 V, under the 30 V maximum), M12 cordset (sold separately), shielded cable. Measure the excitation voltage on a spare channel and compute the ratio. |
| **Dwyer RHP** (humidity and temperature) | **Yes, two signals.** Humidity is 4–20 mA, loop powered from 10–35 VDC. Temperature is a separate passive Pt1000 resistance. | See Signal conditioning. |
| **"FM" VTV122** | **Yes, but it is a transmitter.** It is an **ifm** efector VTV122: single axis, 0–25 mm/s RMS, 10–1000 Hz, 4–20 mA, 9.6–32 V two-wire, M12 A-coded connector. | 24 VDC loop supply, shunt resistor, ifm M12 cable. Confirm this is the part you have. |
| **MCC USB-2408-2AO** | Yes. | See DAQ roles. |
| **MCC USB-1604HS-2AO** | Yes. | See DAQ roles. |

## Signal conditioning

- **4–20 mA loops (RHP humidity and VTV122):**
  - Both need a 24 VDC supply.
  - Put a 250 Ω, 0.1 % precision shunt in each loop, giving 1–5 V into a DAQ input.
  - The VTV122 allows at most 720 Ω of load at 24 V, so 250 Ω is fine.
  - Use differential inputs across the shunt.
- **Pt1000 (RHP temperature):**
  - As recalled, the USB-2408 has thermocouple and voltage inputs but **no RTD input**, so it cannot read this directly.
  - Use a Pt1000-to-4–20 mA converter, or a DAQ with RTD support (for example MCC's USB-TEMP).
  - Check the RHP manual for whether the RTD is 2-wire or 3-wire.
- **Thermocouples:** the USB-2408 has built-in cold-junction compensation, so connect them directly to its screw terminals.

## DAQ roles

- **USB-2408 (slow, 24-bit, isolated inputs):** 8 differential inputs (or 16 single-ended), 500 VDC isolation, about 1 kS/s aggregate, 8 digital I/O, 2 counters. Use for thermocouples, loop shunts and string pots. Aggregate 1 kS/s is plenty for slow-moving jacks.
- **USB-1604HS (fast, simultaneous sampling):** only **4** single-ended 16-bit inputs, each with its own ADC, up to 1.33 MS/s per channel, ±10 / ±2.5 / ±0.5 V, **not isolated**, needs an external 5 V supply. It also has 4 counters (3 with quadrature) and 32 digital lines. Reserve its 4 channels for dynamic signals such as motor current or vibration waveforms.
- **Channel count is the real limit.** With several jacks and a string pot, current transducer, vibration sensor and thermocouple per jack, the 8 channels on the USB-2408 fill quickly. See [Four_Jack_Characterization_Plan.md](./Four_Jack_Characterization_Plan.md) for a full channel allocation.
- **Sync between DAQs:** the two devices do not share a hardware clock. Align data by software timestamps.

## Gaps

1. **Load.** There is no load cell. Load per jack and load sharing are the most useful measurements for this application. Use a bridge conditioner with a voltage or 4–20 mA output.
2. **Speed and limit switches.** There is no motor speed or encoder and no limit-switch status. The DAQs have digital inputs and counters you can use.
3. **Vibration detail.**
   - The VTV122 gives an RMS value only, with no waveform or spectrum.
   - Its 10 Hz floor will miss slow shaft or ball-screw faults.
   - A spectrum needs an IEPE accelerometer and a conditioner.
4. **Current on a VFD.** Drive output is noisy, so use true-RMS transducers rated for it. The MA63 has only a 60 A range, so confirm motor currents fit.
5. **Accuracy.** The string pot's ±0.35 % of full scale is about ±0.28 in over 80 in. Check that this is acceptable for jack-to-jack positioning.
6. **Environment.**
   - Below ground means condensation. USB DAQs are IP20 and rated non-condensing, so they need a NEMA 4X enclosure with desiccant, or must sit above ground.
   - The RHP and VTV122 are rated IP66 and above.
   - The RHP accuracy figure is for 20–80 % RH at 25 °C, and condensation affects it.
7. **USB distance.** USB runs about 5 m. For a pit, use an active or fiber USB extender, or place the DAQ in the pit and run Ethernet or fiber to the PC.
8. **Noise and grounding.** Use shielded cable grounded at one end, keep signal cables away from motor power, and keep a single ground reference.
9. **Safety.** This setup should only monitor. Do not rely on it for interlocks or e-stop.
10. **Security.** If this is a secure installation, plan for restricted access, an isolated network and protected log storage.

## LabVIEW and the "plug in and go" goal

- **Driver support:** MCC devices use MCC's Universal Library, which has LabVIEW support. NI-DAQmx will not see them. Confirm the library works with your LabVIEW version and bitness, and install it with Quick .
- **Stable setup:**
  - Identify each DAQ by serial number, not board number.
  - Keep scaling (shunt value, 4–20 mA span, string-pot span, thermocouple type) in a config file, one row per channel.
  - Use a powered industrial USB hub.
- **Software structure:**
  - Producer/consumer loops.
  - Log to TDMS.
  - Alarms on temperature, humidity, current and vibration.
  - A manual-entry screen for the Fluke and Extech readings so spot checks sit alongside logged data.

## Items to confirm in the MCC datasheets

- Whether the USB-2408 has an RTD input (it appears not; thermocouple and voltage only. MCC's USB-TEMP family supports RTDs).
- The input counts, sample rates and isolation quoted above come from distributor and search summaries, not from the MCC datasheets directly. Confirm them.
- The analog-output current limit (about ±5 mA per summaries; relevant if exciting the string pots from it).

## Datasheet links

- Fluke 62 MAX: <https://www.labsps.com/wp-content/uploads/2023/12/Fluke62MAX_Datasheet.pdf>
- Extech MA63: <https://www.extech.com/ma63> and <https://www.tequipment.net/Extech/MA63/Clamp-Meters/>
- Celesco SG1: <https://www.primatec.no/image/catalog/Celesco/sg1.pdf> and <https://www.mouser.com/datasheet/2/418/6/ENG_DS_SG1_A-1134682.pdf>
- Dwyer RHP: <https://dwyer-products.de/wp-content/uploads/2023/05/RHP.i_2024.pdf>
- ifm VTV122: <https://www.ifm.com/us/en/product/VTV122> and <https://www.instrumart.com/assets/ifm-efector-vtv-series-VTV122-vibration-transmitter-datasheet.pdf>
- MCC USB-2408-2AO: <https://files.digilent.com/manuals/USB-2408-2AO.pdf>
- MCC USB-1604HS-2AO: <https://files.digilent.com/manuals/USB-1604HS-2AO.pdf>

## Next steps

Provide the number of jacks and the measurements wanted per jack. From that, a channel map, bill of materials (shunts, supply, cables, enclosure) and LabVIEW channel-config template can be produced.
