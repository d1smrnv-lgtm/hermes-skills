---
name: arduino-uno-projects
description: "Use when building Arduino UNO projects on macOS."
version: 1.0.0
author: Hermes Agent
license: MIT
dependencies: []
platforms: [macos]
metadata:
  hermes:
    tags: [arduino, uno, macos, hardware, electronics, games]
    related_skills: []
---

# Arduino UNO projects on macOS

## When to use
- User has an Arduino UNO (or compatible clone) connected to a Mac.
- Need to detect the board, compile/upload a sketch, plan pins, or build a small interactive project (games, LEDs, buttons, buzzers).

## Tooling on this machine
- Arduino IDE app: `/Applications/Arduino IDE.app`
- Bundled CLI: `/Applications/Arduino IDE.app/Contents/Resources/app/lib/backend/resources/arduino-cli`
- Installed core: `arduino:avr` (version 1.8.8)
- Board FQBN for UNO: `arduino:avr:uno`

## Detect the board and port
1. List serial ports: `ls /dev/cu.*`
2. Genuine UNO usually appears as `/dev/cu.usbmodem*`.
3. FTDI-based clones (common in starter kits) appear as `/dev/cu.usbserial-*` and show as `FT232R USB UART` in `ioreg`.
4. `arduino-cli board list` may report the port as `Unknown`; do not rely on auto-detection. Pass `--fqbn arduino:avr:uno` explicitly.

## Standard workflow
1. Create a sketch directory whose name matches the `.ino` file, e.g. `mkdir -p ~/arduino/my_sketch`.
2. Write the sketch to `~/arduino/my_sketch/my_sketch.ino`.
3. Compile:
   `arduino-cli compile --fqbn arduino:avr:uno ~/arduino/my_sketch`
4. Upload:
   `arduino-cli upload -p /dev/cu.usbserial-XXXX --fqbn arduino:avr:uno ~/arduino/my_sketch`
5. Confirm success from CLI output (`New port upload` / no avrdude errors).
6. For quick LED tests, use `LED_BUILTIN` (pin 13) and `digitalWrite`.

## Pin-planning rules
- `D0`/`D1` are Serial RX/TX; avoid using them as GPIO when uploading or debugging over USB.
- Analog pins `A0`-`A5` can be used as digital pins when needed.
- Buttons: use `INPUT_PULLUP` and connect the other leg to `GND`; no external resistor needed.
- LEDs: always use a 220 Ω resistor in series; cathode to `GND`.
- 7-segment displays need a resistor per segment (220 Ω). Determine common cathode vs common anode before coding; provide a `COMMON_ANODE` flag and invert outputs if needed.
- Avoid depending on pin 13 for clean output during upload; the bootloader blinks it.

## Deliverable preferences
- When building a project for a child (~10 years), keep rules short and concrete.
- Provide three artifacts together:
  1. The `.ino` sketch file.
  2. A visual connection guide (HTML/SVG preview is acceptable).
  3. A downloadable spreadsheet table (XLSX) with pin mapping, rules, components, and build steps.
- If the user asks for a picture, provide the HTML/SVG preview plus an XLSX and, when possible, a screenshot of the rendered guide.
- Prefer using as many kit components as practical, but do not force a component if it creates pin conflicts.
- When the kit has limited wire counts, include a wire inventory (M-M/F-F) and mark which connections use the breadboard rails.
- When the user asks for a picture, render the HTML/SVG guide to PNG via browser screenshot and provide it as MEDIA.
- Build guides: show all wire runs first (signal, power, ground), then button placement, then remaining components.
- When the user asks for a picture, include per-connection wiring diagrams (Arduino pin → component) in addition to the overall pin map.
- Keep visuals concrete and simple for a ~10-year-old; avoid dense schematics without labels.

## Common pitfalls
- Auto-detection fails: pass FQBN explicitly.
- Forgetting resistors on LEDs/7-segment segments can damage components or make them dim.
- Mixing common-anode and common-cathode logic produces an inverted or blank display.
- Button bounce: wait for release (`while (digitalRead(pin) == LOW) {}`) and add a small delay.
- 7-segment on `D0`/`D1` can interfere with serial upload; prefer `D2`-`D8`.
- FTDI clones report as `Unknown` in `arduino-cli board list`; do not wait for auto-detection, pass `/dev/cu.usbserial-*` and `--fqbn arduino:avr:uno` explicitly.
- `D13` is toggled by the bootloader during upload/reset; usable for buttons/inputs, but avoid for outputs that must stay clean.
- Colored kit buttons may not match LED colors; map by the user's stated pairing (e.g. red→red, white→yellow, blue→green) and keep `btnPins[i]` aligned with `ledPins[i]`.

## Related reference
- `references/two-player-color-reaction-game.md` — a complete two-player reaction game using 3 LEDs, 6 buttons, buzzer, 7-segment, and potentiometer.
