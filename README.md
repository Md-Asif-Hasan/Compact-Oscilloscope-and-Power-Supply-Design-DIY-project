Compact Oscilloscope + 0–30V/3A PSU (EEE316 Group 07)
This repository contains schematics, PCB assets, firmware, and documentation for:

A single‑channel compact oscilloscope (0–200kHz analog bandwidth class, 12‑bit display scaling, selectable time/div and volts/div, trigger modes).

A linear adjustable bench power supply with 24VAC input, 0–30V output, 2mA–3A current limit, and low ripple.

The project was designed, simulated, fabricated, assembled, and tested as part of EEE316.

Repository Structure
hardware/

oscilloscope/

schematics/ Schematic PDFs and source files

pcb/ PCB files (Gerbers, drills, assembly drawings)

bom/ BOM CSV with MPNs and footprints

psu/

schematics/

pcb/

bom/

firmware/

oscilloscope_arduino_oled/ Arduino sketch for SSD1306 OLED scope

oscilloscope_stm32_tft/ STM32F103 + ILI9341 TFT implementation (src, build files)

simulation/

psu_ltspice/ Loop, ripple, and transient sims

afe_ltspice/ Attenuator + anti-alias filter + step response

docs/

report/ Project report and slides

bringup_guides/ Assembly and bring-up notes

calibration/ Procedures and data sheets for calibration

If a directory is missing in the initial commit, it will be added as assets are finalized.

Features
Oscilloscope

Input: up to 50Vpk with 10× probe; selectable 1×/10× path

Timebase: 50ms/div to 200µs/div (Arduino OLED build) with trigger rising/falling

Display: 0.96" SSD1306 OLED or 2.4" ILI9341 TFT

Controls: SELECT, UP, DOWN, HOLD

Storage: settings persisted in EEPROM/flash

Optional: STM32 version with higher performance rendering and USB CDC

Power Supply

Input: 24VAC transformer (>90W for 30V/3A)

Output: 0–30V adjustable, 2mA–3A current limit

Architecture: bridge rectifier + bulk filter + TL081 op‑amp loops + pass transistor

Indicators: LED status, panel pots for V and I

Protections: input fuse, short-circuit handling, heat sinking

Getting Started
Prerequisites

Ubuntu 22.04+ (recommended)

KiCad 7/8 (to open/edit PCB and schematics)

Arduino IDE 2.x (for OLED oscilloscope build)

ARM GCC toolchain / STM32CubeIDE (for STM32 TFT build, optional)

Python 3.10+ (utility scripts)

Install Python libs:

pip install -r requirements.txt

Build: Oscilloscope (Arduino + SSD1306)
Hardware

Assemble the OLED scope PCB and connect buttons to D8–D11, INT0 on D2, analog in on A0, and an optional 1×/10× selector.

Power from 9–12V DC jack or regulated 5V as per your board.

Firmware

Open firmware/oscilloscope_arduino_oled/_20190212_OLEDoscilloscope.ino in Arduino IDE.

Install libraries: Adafruit GFX, Adafruit SSD1306, EEPROM (bundled).

Board: Arduino Uno (ATmega328P) or compatible at 16MHz, 5V.

Compile and upload.

Calibration

Connect the built-in test square wave (or a 1kHz, ~1Vpp generator).

Adjust trimmers (if present) and perform probe compensation.

Save settings; ranges persist in EEPROM.

Build: Oscilloscope (STM32 + TFT, optional)
Hardware

Assemble STM32F103C8 MCU, ILI9341 SPI TFT, buttons, BNC input, op-amp AFE if used.

Firmware

Use STM32CubeIDE or GNU Arm Embedded Toolchain.

Configure SPI DMA for display, timers/ADC/DMA for sampling, and GPIO for UI.

Build and flash via SWD.

Build: Power Supply
Hardware

Use 24VAC transformer (>90W recommended for 30V/3A).

Assemble bridge (1N5408s), bulk capacitors (e.g., 3300–4700µF), TL081-based control loops, pass devices (9014/9015 per schematic), shunt (0.47Ω/5W), pots, and output terminals.

Provide adequate heat sinking and ventilation.

Bring‑up

First power with a current‑limited bench supply on the DC bus if available.

Verify reference rails, op‑amp outputs, and current‑limit loop at low voltage.

Load test with dummy loads; measure ripple with a scope using a bandwidth limit and short ground spring.

Simulations
PSU: ripple vs load, step load transient, stability margins (phase/gain).

AFE: Bode and time-domain step to confirm anti‑alias corner and settling.

Files under simulation/ (LTspice or Proteus equivalents).

Testing and Calibration
Oscilloscope

Short input to measure noise floor.

1kHz sine at 1Vpp to verify scaling and trigger stability.

Timebase check against a known 1MHz/100kHz reference if available.

Power Supply

No‑load voltage sweep 0–30V.

Load regulation at 0.1A, 0.5A, 1A, 2A, 3A.

Ripple measurement and thermal soak at 50–75% load.

Safety
Do not connect mains directly to the oscilloscope input.

The PSU primary wiring must be insulated and fused; observe local electrical safety standards.

Discharge capacitors before handling; use proper heat sinks on pass devices.

Roadmap
Multi‑channel oscilloscope option (2‑ch)

Advanced triggering and on‑device FFT

Digital panel meters for PSU with UART logging

USB data streaming and PC client

License
Educational project. If redistributing, include attribution to the original student team and respect third-party library licenses.
