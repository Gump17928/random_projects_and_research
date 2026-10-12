# Meeting Prep: Screw Jack Monitoring

Companion to [Four_Jack_Characterization_Plan.md](./Four_Jack_Characterization_Plan.md) and [Screw_Jack_Sensor_Assessment.md](./Screw_Jack_Sensor_Assessment.md).

## Data files

These CSV files in [data/](./data) render as tables on GitHub and open in Excel or Google Sheets. Each has status and owner columns to track progress.

| File | Contents |
|---|---|
| [open_questions.csv](./data/open_questions.csv) | 43 questions to ask, with why each matters and which decision it affects. Blank answer, owner and date columns. |
| [sensors_and_interfaces.csv](./data/sensors_and_interfaces.csv) | Every sensor and DAQ, how it connects, the gap, and the recommended action |
| [channel_map.csv](./data/channel_map.csv) | Every proposed channel (57), with device, channel, scaling and rate |
| [bom_and_components.csv](./data/bom_and_components.csv) | Everything needed to make this a complete system, with status and the question each item depends on |
| [readiness_checklist.csv](./data/readiness_checklist.csv) | Checks to run so the system works when plugged in |

## Questions to be ready to ask

The full list is in [open_questions.csv](./data/open_questions.csv). The ones that block the most decisions:

1. **Purpose:** what decisions will this data support, and is this a short campaign or permanent monitoring? (Q01, Q02)
2. **Jacks and load:** jack model, rating, ratio, lead, object weight, and are the jacks independent or ganged? (Q06, Q07, Q08)
3. **Load cell mounting:** where can load cells go, and can the equipment be modified? (Q09)
4. **Motors and drives:** motor nameplate data, and is there a VFD that can share current, torque and speed data over a network? (Q13, Q14)
5. **Control signals:** what voltage are the run, direction and limit signals, and can we tap them? (Q15)
6. **Pit conditions and access:** temperature, humidity, condensation, water, access rules, and where an enclosure can go. (Q19, Q20)
7. **Distance and IT:** how far is the PC from the pit, and what do site security and IT allow (laptops, USB devices, networking)? (Q21, Q23)
8. **Data needs:** do we need transients and vibration spectra, or only trends? (Q27)
9. **Inventory:** exactly what do we own (string pots, VTV122, RHP, cables, supplies)? Is the VTV122 the ifm transmitter? (Q32, Q33)
10. **Budget, schedule and installers:** budget, timeline, who installs and approves. (Q40, Q41)

## Better ways to measure what cannot be plugged into a laptop

Two of your tools have no data output at all: the Fluke 62 MAX and the Extech MA63. The VTV122 and RHP are not handheld, but they are 4–20 mA devices and need supporting hardware.

**Temperature (instead of the Fluke 62 MAX)**
- **Best:** surface-mount thermocouples read by the USB-2416. They are cheap, give continuous logs and are already in the plan.
- **If non-contact is required:** a fixed-mount IR sensor with a 4–20 mA output. It needs the same shunt and 24 V supply as the other loops.
- **For portable surveys:** check whether a logging IR thermometer or thermal camera with USB, Bluetooth or app export fits your needs. Verify the model before buying.
- **Keep the Fluke** for spot checks and for verifying the thermocouples.

**Current (instead of the Extech MA63)**
- **Best:** a Modbus 3-phase power meter on each motor, giving voltage, current per phase, kW, PF and kWh.
- **If there is a VFD:** read current, torque and speed from the drive itself over Modbus or Ethernet.
- **For fast capture:** split-core true-RMS transducers with a 0–10 V output on the USB-1604HS.
- **Other option:** clamp meters with Bluetooth or USB data output exist. Verify the model and logging rate before relying on one.
- **Keep the MA63** as an independent cross-check, but confirm that motor currents are within its 60 A range.

**Vibration (VTV122 limits)**
- It gives RMS velocity only. For spectra use IEPE accelerometers and a conditioner (Phase 2).

**Last resort for anything handheld:** a manual entry panel in LabVIEW that records the reading with a timestamp. This is useful for spot checks but should not be the main data path.

## Will this need USB, or something similar?

Yes. Plan on **USB as the main link**, with a few exceptions.

- **MCC DAQs (USB-2408, USB-2416, USB-1604HS, auxiliary):** USB. Some need an external 5 V supply. Use a powered industrial USB hub and identify each DAQ by serial number.
- **Power meters (Modbus RTU):** RS-485, connected to the laptop through an isolated RS-485 to USB adapter. Modbus TCP over Ethernet is possible if the meters or drive support it.
- **Sensors (string pots, 4–20 mA loops, thermocouples, load cells, current transducers):** these do not connect to a laptop directly. They wire to the DAQs through the conditioning described in the plan.
- **Handhelds (Fluke, Extech):** no link. Manual entry only.

Practical limits to plan around:

- **USB cable length is about 5 m.** For a pit, use an active USB extender or fiber link, or place the DAQs in a pit enclosure with a single Ethernet-based link to the PC. MCC also makes Ethernet-based DAQs, which avoid USB range limits, but they are different models from what you own.
- **Environment:** USB DAQs are IP20 and non-condensing. They belong in a sealed enclosure with desiccant, or above ground.
- **Laptop settings:** disable sleep and USB selective suspend, and consider a UPS. A pre-configured laptop (InstaCal, ULx for LabVIEW, config file) is what makes the setup plug and go.
- **One-cable goal:** terminate all field wiring at one enclosure. The laptop then connects with one USB trunk (through the powered hub) and one power cable.
- **LabVIEW compatibility:** MCC devices use ULx for LabVIEW (versions about 2022 to 2025) and InstaCal. They are not visible to NI-DAQmx or NI-MAX.

## Viewing and sharing online

The folder is not a Git repository. To share it on GitHub, create a repository and push these files. GitHub renders the CSV files as tables and the Markdown files as formatted pages. The CSVs also open directly in Excel or Google Sheets for editing answers during the meeting.
