# Smart IV Drip Monitor — Arduino Uno R3 port

**Simulation / educational prototype only. This is not a medical device and must not be connected to or used to control a real IV infusion.**

## Files

- `sketch.ino` — converted Arduino sketch.
- `diagram.json` — Wokwi circuit diagram.

## Uno R3 pin map

| Uno pin | Connection | Purpose |
|---|---|---|
| A0 | Flow potentiometer, Bed 1 | Flow input |
| A1 | Flow potentiometer, Bed 2 | Flow input |
| A2 | Flow potentiometer, Bed 3 | Flow input |
| A3 | Alert potentiometer, Bed 1 | Alert threshold input |
| A4 | Alert potentiometer, Bed 2 | Alert threshold input |
| A5 | Alert potentiometer, Bed 3 | Alert threshold input |
| D2 | 74HC595 #1 DS | Serial data to chained output registers |
| D3 | Both 74HC595 SHCP + 74HC165 CP | Shared shift clock |
| D4 | Both 74HC595 STCP | Output latch |
| D5 | 74HC165 PL | Parallel-load control, active LOW |
| D6 | 74HC165 Q7 | Button data back to Uno |
| D9 | Servo PWM | Priority pointer servo |
| D10 | Buzzer + | Alarm output |
| D0/D1 | USB Serial | Leave unconnected to other parts |

## 74HC595 output allocation

**First 74HC595 (nearest the Uno):**

- Q0: Bed 1 green LED
- Q1: Bed 1 red LED
- Q2: Bed 2 green LED
- Q3: Bed 2 red LED
- Q4: Bed 3 green LED
- Q5: Bed 3 red LED
- Q6: LCD RS
- Q7: LCD E
- Q7S connects to DS on the second 74HC595.

**Second 74HC595:**

- Q0 → LCD D4
- Q1 → LCD D5
- Q2 → LCD D6
- Q3 → LCD D7

Both 595s: VCC → 5V, GND → GND, OE → GND, MR → 5V. Connect both SHCP pins to Uno D3 and both STCP pins to Uno D4.

## 74HC165 button allocation

- D0, D1, D2 → START buttons for Beds 1, 2, 3.
- D3, D4, D5 → BAG SIZE buttons for Beds 1, 2, 3.
- D6 and D7 → 5V (unused inputs held HIGH).
- PL → Uno D5; CP → Uno D3; Q7 → Uno D6.
- CE → GND; DS → GND; VCC → 5V; GND → GND.
- Each used input gets a 10 kΩ pull-up to 5V. Its pushbutton connects that input to GND, so a pressed button reads LOW.

## LCD wiring

Use a **standard parallel 20×4 LCD**, not the I²C version:

- RS ← first 595 Q6; E ← first 595 Q7.
- D4–D7 ← second 595 Q0–Q3.
- RW → GND; VSS → GND; VDD → 5V; V0 → GND for the Wokwi simulation.
- A → 5V; K → GND.
- LCD D0–D3 are unused in 4-bit mode.

## Other wiring

- Each LED is driven through its own 220 Ω series resistor; LED cathodes go to GND.
- Each potentiometer: VCC → 5V, GND → GND, SIG → its assigned analog input.
- Servo: PWM → D9, V+ → 5V, GND → GND.
- Buzzer: positive pin → D10, other pin → GND.
- All grounds must be common.

## Controls and Serial Monitor

- Set each bed's FLOW dial and ALERT % dial.
- A bag is detected from its flow input; press that bed's START button to begin monitoring.
- Tap its SIZE button to cycle 250 mL → 500 mL → 1000 mL.
- Short START press: start/new bag. Long START press: acknowledge active alarms. Holding all three START buttons: reset monitoring state.
- Serial Monitor baud rate: **115200**.
- Serial commands: `h` history, `a` acknowledge, `r` reset, `t` cycle time speed, `d` demo volumes, `p` simulated power cut/restart, `?` help.

## Suggested Wokwi test sequence

1. Start simulation; verify the LCD splash screen, LED self-test, and servo sweep.
2. Turn up each bed's FLOW dial and ALERT dial; verify all three beds are detected and show `SET A%` until the alert dial is set.
3. Press each START button; verify the selected bag volume, green LED, LCD bed row, and Serial dashboard.
4. Change each SIZE button and verify the bag size changes immediately before START, or applies to the next bag after START.
5. Change a flow dial substantially; verify the movement filter and warning/critical states. Test the alert percentage dial by lowering the displayed remaining percentage below its selected threshold.
6. Test short press, long press, and the three-button reset hold.
7. Send `h`, `t`, `d`, and `p` from Serial Monitor; after `p`, confirm the saved state is restored.

## Porting notes

The original Mega code's monitoring rules, three-bed model, flow/alert calculations, alarm escalation, servo priority, EEPROM rotation, event history, Serial commands, and dashboard text are retained. The hardware interface was changed: the I²C LCD was replaced with a parallel 20×4 LCD driven through two chained 74HC595s; the six LEDs use the first 595; and the six buttons are read through one 74HC165. This avoids using A4/A5 for I²C and leaves all six analog inputs available for the original potentiometers.

The sketch was syntax-checked with a local Arduino API stub, but it has not been run in Wokwi or compiled with the official AVR toolchain in this environment. Please run the simulation and follow the test sequence above before relying on the result even as a demonstration.
