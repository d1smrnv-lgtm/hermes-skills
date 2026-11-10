# Two-player color reaction game (Arduino UNO)

## Concept
Arduino lights one of three colors (red/yellow/green). Two players each have matching color buttons. First correct press scores a point. First to 9 wins.

## Components
- 3 LEDs (red, yellow, green) + 3x 220 Ω
- 6 buttons
- Active buzzer
- 1-digit 7-segment display + 7x 220 Ω
- Potentiometer 10 kΩ

## Pin map
- 7-segment segments a-g: D2-D8
- Player 1 buttons: D9 (red button), D10 (white button), D11 (blue button)
- Player 2 buttons: D12 (red button), D13 (white button), A0 (blue button)
- LEDs: A1 (red), A2 (yellow), A3 (green)
- Buzzer: A4
- Potentiometer (middle pin): A5

## Button color mapping
- Red button -> red LED (index 0)
- White button -> yellow LED (index 1)
- Blue button -> green LED (index 2)
- Keep the button index aligned with the LED index in code (`btnPins[i]` drives `ledPins[i]`).

## Rules
1. Arduino lights one color.
2. Both players press the matching color button as fast as possible.
3. First correct press scores +1.
4. Wrong color gives an error beep and no point.
5. 7-segment shows the leader's score.
6. First to 9 wins; play victory animation.
7. Potentiometer sets round speed (about 300-1500 ms).

## Implementation notes
- Use `INPUT_PULLUP` for all buttons.
- Use `randomSeed(analogRead(pot) + millis())`.
- Keep a 3-second timeout per round.
- After a correct press, blink the target LED before clearing.
- Provide a `COMMON_ANODE` flag for the 7-segment display.

## Wire inventory (this kit)
- 13x male-male (папа-папа): D2-D8 (7-segment), D9-D13 and A0 (buttons).
- 4x female-female (мама-мама): A1-A3 (LEDs), A4 (buzzer).
- GND, 5V, and A5 (potentiometer) use the breadboard power rails (no extra wires).

## Display type
- The kit's 1-digit 7-segment is common anode; set `COMMON_ANODE true` in the sketch.

## Build guide
1. Run all wires first: signal lines (D2-D8, D9-D13, A0-A5), then 5V and GND.
3. Add LEDs with 220 Ω resistors, buzzer, 7-segment with 220 Ω per segment, and potentiometer.
4. Note: D13 is toggled by the bootloader during upload/reset; it is fine for a button input.
