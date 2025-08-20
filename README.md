# README.md — Compact Oscilloscope (Arduino Cloud, Arduino Uno) + 0–30V/3A Linear PSU

This repository hosts the designs, firmware, and documentation for two lab instruments built for EEE316:
- A single‑channel compact oscilloscope implemented on Arduino Cloud using an Arduino Uno and a 0.96" SSD1306 OLED (optional TFT variant referenced in docs).
- A linear adjustable power supply with 24VAC input and 0–30V, 2mA–3A output capability.

The oscilloscope firmware and project configuration target Arduino Cloud as the primary development and deployment platform. Local Arduino IDE builds are still possible.

## Repository layout

- hardware/
  - oscilloscope/
    - schematics/           Oscilloscope schematic PDFs
    - pcb/                  Board files, Gerbers, drills, assembly drawings
    - bom/                  Bill of Materials (CSV with MPNs/footprints)
  - psu/
    - schematics/           PSU schematic PDFs (bridge, filters, TL081 loops)
    - pcb/                  Gerbers, drills, assembly
    - bom/                  BOM CSV
- firmware/
  - oscilloscope_arduino_cloud/   Sketch source used in Arduino Cloud (mirrors oscilo_code.txt)
  - oscilloscope_arduino_oled/    Local build folder for Arduino IDE users
- simulation/
  - psu/                   Ripple/loop/transient sims (LTspice/Proteus models)
  - afe/                   Attenuator + anti‑alias filter sims
- docs/
  - report/                Project report, slides, images
  - bringup_guides/        Assembly, wiring, calibration notes
  - calibration/           Procedures and data logs

If a folder is missing initially, it will be added with the next commit.

## Oscilloscope — Overview

- Platform: Arduino Uno (ATmega328P, 16MHz, 5V) deployed via Arduino Cloud.
- Display: 0.96" SSD1306 I2C OLED (128×64).
- Input: A0 analog input with optional 1×/10× attenuation selection; recommended use with 10× probe up to 50Vpk.
- Controls:
  - D8 SELECT, D9 UP, D10 DOWN, D11 HOLD (internal pull‑ups)
  - D2 external interrupt for the button handling ISR
- Features:
  - Timebase: 50ms/div to 200µs/div (discrete ranges)
  - Volts/div ranges with automatic scaling modes
  - Rising/falling edge trigger and HOLD
  - EEPROM‑backed settings (ranges/trigger persist across power cycles)
- Sampling: Timer + analogRead() scheduling with prescaler changes per range.
- Notes: This is an educational scope (≈0–200kHz class). For higher bandwidth, consider an external ADC/STM32 variant.

## Power Supply — Overview

- Input: 24VAC transformer (>90W for full 30V/3A capability)
- Topology: Full bridge rectifier (1N5408) + bulk capacitor + linear regulation using TL081 op‑amps and pass transistor network (9014/9015).
- Output: 0–30V adjustable, current limit 2mA–3A, ripple target ≈0.01% with proper layout/filters.
- UI: Front‑panel potentiometers for voltage/current, status LED, binding posts.
- Protections: Fuse, short‑circuit behavior tuned by current‑limit loop, adequate heat sinking required.

## Bill of Materials (abridged)

Oscilloscope
- Arduino Uno (ATmega328P)
- SSD1306 0.96" OLED (I2C, 0x3C)
- Tact switches ×4, LED (optional)
- Resistor network for attenuation and biasing as per schematic
- Optional: 10× probe, BNC/SMA input connector, TVS/ESD parts

PSU
- 24VAC transformer (≥90W for 30V/3A)
- 1N5408 diodes (bridge)
- Electrolytics (e.g., 3300–4700µF/50V), film caps
- TL081 op‑amps, small‑signal transistors 9014/9015
- Shunt resistor 0.47Ω/5W
- Pots for V/I adjust, binding posts, fuse, heatsinks

See hardware/*/bom for full part lists.

## Hardware wiring (Oscilloscope)

- OLED I2C: SDA → A4, SCL → A5, VCC → 5V, GND → GND
- Buttons: SELECT→D8, UP→D9, DOWN→D10, HOLD→D11 (to GND on press; relies on internal pull‑ups)
- INT: D2 configured as external interrupt for button state capture
- Signal input: to A0 via the front‑end network (divider/attenuator/ESD, as per schematic)
- Optional 1×/10× control signal: D12 used as output/Hi‑Z per code to emulate path selection

## Develop and deploy (Arduino Cloud)

1) Create a new Arduino Cloud Thing
- Add a Sketch and copy the oscilloscope code from firmware/oscilloscope_arduino_cloud (or oscilo_code.txt).
- Board: Arduino Uno.
- Libraries (add via Library Manager in Arduino Cloud):
  - Adafruit GFX Library
  - Adafruit SSD1306
  - EEPROM (bundled with AVR core)

2) Variables/Secrets
- No cloud variables required for the basic build.
- Ensure I2C address 0x3C matches your OLED module.

3) Compile and Upload
- Connect the Uno over USB.
- From Arduino Cloud Web Editor, Verify → Upload.
- On first boot the sketch initializes EEPROM defaults; settings persist thereafter.

## Local build (optional, Arduino IDE)

- Open firmware/oscilloscope_arduino_oled/_20190212_OLEDoscilloscope.ino
- Board: Arduino Uno
- Install libraries:
  - Adafruit GFX
  - Adafruit SSD1306
- Compile and upload via USB.

## Calibration and use (Oscilloscope)

- Power: 9–12V DC input to the board or 5V via USB (match your assembly’s power routing).
- Probe compensation: Use a 1kHz square wave source. Adjust trimmers if present; verify flat tops and sharp edges.
- Timebase/Volts: Use SELECT to choose field, UP/DOWN to adjust; HOLD freezes display.
- Safety: With 10× probe, limit to ≤50Vpk; never connect to mains directly; use isolation and proper grounding.

## PSU bring‑up checklist

- First power the low‑voltage control section from a current‑limited bench supply if available.
- Verify reference and op‑amp rails, then connect the 24VAC transformer.
- Sweep the voltage setpoint with no load; verify current‑limit action using a dummy load.
- Measure ripple with a scope using a short ground spring and bandwidth limit.

## Known limitations

- Uno ADC sampling and OLED refresh cap total throughput; rapidly changing high‑frequency signals will alias.
- PSU thermal performance depends on heatsink sizing and enclosure airflow.
- Layout and wiring quality strongly affect noise/ripple; follow the provided placement notes.

## Roadmap

- Optional STM32 + TFT path with DMA rendering and faster acquisition
- USB serial waveform dump and PC plotting script
- Digital panel meter and UART logging for PSU

## License

Educational use. Include attribution to the original student team and honor third‑party library licenses.
