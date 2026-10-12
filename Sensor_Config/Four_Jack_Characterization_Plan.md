# Four Screw Jack Characterization Plan

Goal: measure as much as practical per jack to characterize load, current functionality and efficiency of the existing system, and to support improvement and future design decisions.

Builds on [Screw_Jack_Sensor_Assessment.md](./Screw_Jack_Sensor_Assessment.md).

## Assumptions to confirm

These drive sensor choice. Please confirm or correct them.

- Four jacks, each with its own motor, all mechanically independent (not ganged by a drive shaft).
- Motors are 3-phase AC. It is not yet known whether they run direct-on-line or from a VFD.
- Travel is 80 in or less (the SG1-80 string pot range).
- Jack rated capacity, gear ratio, screw lead and object weight are known or can be found.
- One PC runs LabVIEW (2022–2025) with MCC InstaCal and the ULx library.
- Monitoring only. Nothing here controls or interlocks the jacks.

## Measurements per jack

| # | Quantity | Sensor | Signal into the system | Suggested rate | What it tells you |
|---|---|---|---|---|---|
| 1 | Ram position | Celesco SG1-80-3 string pot (owned) | Voltage divider, slow DAQ | 10 Hz | Position, speed, travel time, jack-to-jack skew and platform tilt |
| 2 | Load | Load cell (compression under the jack or inline with the ram) plus bridge conditioner with 0–10 V or 4–20 mA output | Voltage or 4–20 mA, slow DAQ | 10 Hz | Load per jack, load sharing, load vs position, breakaway and binding |
| 3 | Motor electrical power | 3-phase power meter with Modbus RTU (RS-485), or the VFD's own Modbus data if there is a drive | RS-485 to USB | 1 Hz or faster | Voltage, current per phase, kW, kVA, PF, THD, kWh per cycle, phase imbalance |
| 4 | Motor current, fast | Split-core true-RMS current transducer, one phase, 0–10 V output | Voltage, USB-1604HS | 5–10 kS/s | Starting inrush, ripple during travel, stall detection |
| 5 | Input shaft speed | Inductive proximity sensor on a key or gear tooth, or an encoder on the motor | Pulse to counter | counter | Input turns vs ram travel, giving efficiency, lost motion and slip |
| 6 | Gearbox temperature | Thermocouple (K or T), surface mount | Thermocouple, slow DAQ | 1 Hz | Gear and bearing heating, thermal limits per duty cycle |
| 7 | Motor frame temperature | Thermocouple | Thermocouple, slow DAQ | 1 Hz | Motor heating vs load and duty |
| 8 | Screw nut or bearing housing temperature | Thermocouple | Thermocouple, slow DAQ | 1 Hz | Friction heating at the screw |
| 9 | Vibration velocity | ifm VTV122 on the gearbox | 4–20 mA through a 250 Ω shunt, slow DAQ | 10 Hz | Trend of overall vibration (RMS, 10–1000 Hz) |
| 10 | Status signals | Up command, down command, upper limit, lower limit | 24 V digital via opto-isolated inputs | event driven | Marks the start and end of each move, so all data can be cut into cycles |

Shared, not per jack:

| Quantity | Sensor | Notes |
|---|---|---|
| Pit humidity (2 locations) | Dwyer RHP, 4–20 mA through a 250 Ω shunt | Condensation risk |
| Pit temperature (2 locations) | RHP Pt1000 | Needs an RTD-capable input, see allocation |
| String pot excitation voltage | Spare channel | Optional, lets you compute position ratiometrically |
| Spot checks | Fluke 62 MAX, Extech MA63 | Manual entry in LabVIEW, used to cross-check the logged data |

## Derived results you can compute

- **Load vs position:** reveals binding, tight spots and hysteresis between raising and lowering.
- **Load sharing:** compare the sum of the four load cells with the known object weight. Compare each jack to its rating.
- **Platform tilt and skew:** from the four position channels.
- **Efficiency:** mechanical output (load × ram speed) divided by electrical input (kW from the power meter). Only meaningful while raising.
- **Lost motion and slip:** ram travel per input shaft turn compared with the nominal screw lead and gear ratio.
- **Starting and running torque proxy:** motor current and kW vs load and speed.
- **Energy per cycle:** kWh for a full raise and lower.
- **Thermal behavior:** temperature rise per cycle and time to steady state for gearbox, motor and nut.
- **Vibration vs position:** whether vibration peaks at a particular point in travel.
- **Environmental correlation:** humidity and temperature vs friction and temperature rise.
- **Back-driving and holding:** whether a jack creeps when stopped, and holding current if any.

## Channel count and allocation

### Totals for four jacks

| Signal | Count | Type |
|---|---|---|
| Thermocouples | 12 | Slow, isolated differential |
| String pots | 4 | Slow voltage |
| Load cell conditioner outputs | 4 | Slow voltage or 4–20 mA |
| VTV122 loops | 4 | Slow, 4–20 mA via shunt |
| Dwyer RHP humidity loops | 2 | Slow, 4–20 mA via shunt |
| Pt1000 temperature | 2 | RTD |
| Excitation monitor | 1 | Slow voltage, optional |
| Fast motor current | 4 | Fast simultaneous voltage |
| Speed pulses | 4 | Counters |
| Digital status inputs | 16 | 24 V digital via isolation |
| Power meters | 4 | Modbus RTU |

### Proposed devices

| Device | Status | Allocation |
|---|---|---|
| **MCC USB-2416** (16 differential, 24-bit, 500 VDC isolation) | New | 12 thermocouples + 4 string pots. Full at 16 of 16. |
| **MCC USB-2408-2AO** (8 differential, 24-bit, isolated) | Owned | 4 load cell outputs + 4 VTV122 loops. Full at 8 of 8. |
| **MCC USB-1604HS-2AO** (4 simultaneous single-ended, 16-bit, non-isolated) | Owned | 4 fast motor current channels. Its 4 counters take the speed pulses. Its 16 digital inputs take the opto-isolated status signals. |
| **Auxiliary slow DAQ**: MCC USB-TEMP-AI if its voltage inputs suit, otherwise a second USB-2408-2AO plus Pt1000-to-4–20 mA converters | New | 2 Pt1000, 2 RHP humidity loops, 1 excitation monitor, spares |
| **Isolated RS-485 to USB adapter** | New | Modbus to the 4 power meters |

Notes:

- The USB-1604HS has only 4 analog inputs. If you also want vibration waveforms (for spectra), add a second USB-1604HS with IEPE conditioning (Phase 2).
- The 1604HS and the 2416 may need an external 5 V supply. Confirm in the datasheets.
- The 1604HS is not isolated. Use transducers with isolated outputs, and keep a single ground reference.
- The USB-2416 can be expanded with MCC's AI-EXP32 if more channels are needed.
- All channel counts and specifications above come from distributor and search summaries. Confirm against the MCC datasheets before ordering.

## Bill of materials (generic, to be specified)

| Item | Qty | Notes |
|---|---|---|
| Load cells, sealed and IP68 for a pit | 4 | Capacity at or above the jack's rated load. Decide the mounting position, see risks. |
| Bridge conditioners with 0–10 V or 4–20 mA output | 4 | Include excitation |
| String pots SG1-80-3 and M12 cordsets | 4 | Cordset is sold separately |
| ifm VTV122 and M12 A-coded cables | 4 | Check how many you already own |
| Thermocouples, K or T, surface mount | 12 | Plus extension wire and miniature connectors |
| 3-phase Modbus power meters with split-core CTs | 4 | Rated for VFD output if a drive is used. Skip if the VFD provides equivalent data. |
| Split-core true-RMS current transducers, 0–10 V | 4 | For the fast channel |
| Proximity sensors or encoders | 4 | Plus level conversion to 5 V TTL |
| 24 V to 5 V opto-isolated digital input modules | 16 | For status signals |
| 250 Ω, 0.1 % shunt resistors | 6 to 8 | 4 for VTV122, 2 for RHP, plus any extras |
| 24 VDC DIN-rail supply | 1 | Sized for all loops, conditioners and proximity sensors, with margin |
| Regulated 5 V supply | 1 | String pot excitation (4 pots at about 2 mA) and DAQ power |
| NEMA 4X enclosure, desiccant, DIN rail, terminal blocks, fusing, surge protection | 1+ | DAQs are IP20 and non-condensing |
| Shielded twisted-pair cable and glands | as needed | Shield grounded at one end |
| Active USB extender or Ethernet link, powered industrial USB hub | 1 | USB is limited to about 5 m |
| PC with LabVIEW 2022–2025, InstaCal, ULx for LabVIEW | 1 | MCC devices are not visible to NI-DAQmx or NI-MAX |

Optional Phase 2:

- IEPE accelerometers (1 per gearbox, optionally 1 per motor), a 4-channel IEPE conditioner and a second USB-1604HS, for vibration spectra.
- Inline rotary torque transducer on one jack input shaft, for direct torque.
- Inclinometer on the platform.
- Water or leak sensor in the pit.

## LabVIEW architecture

- **Acquisition loops (producer/consumer):**
  - One loop each for the USB-2416, USB-2408 and auxiliary DAQ (slow, about 10 Hz).
  - One continuous streaming loop for the USB-1604HS (fast), with a rolling buffer so startup transients are captured.
  - One Modbus loop for the power meters (1 Hz or faster).
  - One digital status loop, with change events.
- **Identify devices by serial number**, not board number, so the setup survives re-plugging.
- **Channel configuration file:** one row per channel with name, device, channel, sensor type, scaling (shunt value, 4–20 mA span, string pot span, thermocouple type, load cell calibration) and alarm limits.
- **Naming convention** such as `J1_Position_in`, `J1_Load_lbf`, `J1_T_Gearbox_C`, `J1_Vib_mm_s`, `Pit_RH_pct`.
- **Logging:** TDMS, with one group per jack and a shared group for the pit. Log the cycle number and state with every row.
- **Cycle detection:** use the up and down command and limit-switch inputs to split data into cycles automatically, and compute per-cycle metrics (peak load, peak current, travel time, energy, temperature rise).
- **Timestamps:** all devices are timestamped in software. Use the motor start signal as a common marker in both the slow and fast streams.
- **Alarms:** temperature, current, load, humidity and vibration limits. Alarms are advisory only.
- **Manual entry panel** for Fluke 62 MAX and Extech MA63 spot readings, logged with a timestamp.

## Suggested test protocol

1. **Calibration and zero:**
   - Load cells against known weights, or against the known object weight (compare the sum of four).
   - String pots against a tape measure at several points.
   - Thermocouples in an ice bath and against the Fluke 62 MAX.
   - Current against the Extech MA63.
2. **Baseline, no load:** each jack alone through full travel, raising and lowering, several repeats.
3. **Baseline, with object:** all four jacks together, normal operating speed, at least 10 full cycles.
4. **Thermal run:** repeated cycles until temperatures stabilize, to find steady-state and duty limits.
5. **Hold test:** stop at several heights and record load, current and any creep.
6. **Environmental logging:** run the pit humidity and temperature channels continuously over several days, including idle time.
7. **Review:** load sharing, skew, efficiency, lost motion, inrush, thermal rise and vibration trend, per jack.

## Risks and open items

- **Load cell placement:** a load cell under a jack base measures the vertical load including the jack's weight, so tare it. An inline cell changes the mechanical arrangement and must be designed for. Decide this before ordering.
- **Side loads and moments** are not measured by this setup.
- **VFD output:** meters and current transducers must handle PWM waveforms. Reading the drive's own data may be simpler and more useful.
- **String pot accuracy:** ±0.35 % of full scale is about ±0.28 in over 80 in. Calibrate in place.
- **Pt1000 wiring:** confirm 2-wire or 3-wire in the RHP manual.
- **Pit environment:** condensation, water ingress and temperature at the DAQ location.
- **Cable runs and noise:** keep sensor cables away from motor power, use shielded cable, ground one end.
- **Safety:** monitoring only. Do not use this system for interlocks.
- **Datasheet verification:** DAQ specifications are from secondary sources. Confirm in the MCC datasheets before purchasing.

## Information needed to finalize

- Motor nameplate data (voltage, phase, current, power) and whether a VFD is used.
- Jack model, rated capacity, gear ratio and screw lead.
- Object weight and actual travel length.
- Where the PC will sit relative to the pit, and the distance to the pit.
- How many VTV122 and SG1 sensors you already own.
