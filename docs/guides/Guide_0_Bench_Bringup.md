# Guide 0 — Bench bring-up: getting one drive unit moving

**Scope.** Take one belt-driven linear drive unit, a BTT Octopus controller, one stepper driver and one NEMA 17 motor, and turn that motor a known distance under software control. Endstops come in Guide 1. This guide assumes no prior experience with stepper controllers.

**Firmware target.** Marlin 2.1.2.8. Pin names are for the Octopus V1.1 (STM32F446 or F429), from `pins_BTT_OCTOPUS_V1_common.h`. An Octopus Max EZ differs; see §0.11.

**Drivers covered.** TMC2209 over UART and TMC5160 over SPI. Where they differ, both paths appear.

**A note on numbers.** Every value in the body of this guide is generic. Appendix A holds one complete configuration that has been built, flashed and measured on real hardware, for comparison against a new build.

---

## 0.1 Required parts

- BTT Octopus V1.1 controller, and the pinout diagram for the exact board revision in use.
- One stepper driver: TMC2209 or TMC5160, in the plug-in module form.
- One NEMA 17 stepper. The worked numbers assume 1.8°, which is 200 full steps per revolution. A 0.9° motor is 400; see §0.7.
- A 24 V DC supply rated for several amps.
- Motor lead, and power wire rated for the supply current.
- A microSD card formatted FAT32.
- A data-capable USB-C cable and a PC.

---

## 0.2 Understand the board before wiring

The Octopus is a motherboard that takes replaceable driver modules. Five regions matter here.

**Power input.** A screw terminal accepts the main DC supply, which feeds both the motor rails and the logic regulators. A separate heater input exists and stays unused on a motion-only build. Terminal labelling varies between revisions, so confirm polarity against the board's own pinout. The logic needs more than about 14 V, so 24 V is the normal choice.

**Driver slots.** Eight slots, named by motor output. Marlin's default mapping puts MOTOR0 on X, MOTOR1 on Y, MOTOR2 on Z, MOTOR3 on the second Z, and MOTOR4 on the first extruder. This guide uses the X slot, MOTOR0.

**Motor outputs.** Each slot has a 4-pin motor connector. A 4-wire bipolar stepper carries two coils, and each coil takes one pair of pins. Pairing them wrongly is the most common bench mistake; §0.4 covers it.

**USB-C.** Carries data for flashing and for talking to the board. A jumper marked USB-PWR lets the board run its logic from USB alone, which is useful for flashing but leaves the motor unpowered.

**Jumpers.** Three sets matter.

1. *Driver mode jumpers*, under each slot, select STEP/DIR, UART or SPI. The UART pattern differs from the SPI pattern.
2. *Voltage selection*, per driver. A TMC2209, and any TMC5160 that is not a high-voltage part, must sit on the 24 V rail. A non-HV driver on a high-voltage rail is destroyed.
3. *DIAG jumper*, per axis. This connects the driver's stall output to that axis's endstop input. Leave it off for this guide.

---

## 0.3 Set the driver mode jumpers

Disconnect all power first, both the 24 V supply and USB.

1. For a TMC2209, set the slot jumpers to the UART pattern given in the board manual.
2. For a TMC5160, set them to the SPI pattern.

A wrong pattern stops the board configuring the driver. `M122` then returns all zeros or all ones (§0.9).

---

## 0.4 Wire the circuit

> **Disconnect all power before touching the driver or the wiring.** Inserting or removing a driver on a powered board destroys it.

1. Seat the driver in the X slot, aligning its silkscreened labels with the board's.
2. Identify the two motor coils. The datasheet lists them, usually by wire colour. Without a datasheet, use a multimeter on continuity: two wires of one coil read continuity, and wires from different coils read open.
3. Connect each coil pair to one labelled pair on the motor connector.
4. Wire the 24 V supply to the main input, matching polarity to the pinout.
5. Check polarity again before applying power. Reversed polarity destroys the board.

A wrong coil pairing makes the motor buzz, lock or twitch instead of turning. A reversed coil makes it turn the wrong way, which §0.9 corrects in firmware.

---

## 0.5 Set up the toolchain

1. Install VS Code.
2. Download the Marlin 2.1.2.8 source.
3. Open the project folder in VS Code, meaning the folder holding `platformio.ini`, not the inner `Marlin` folder.
4. Accept the prompt to install Auto Build Marlin, which pulls in PlatformIO.

---

## 0.6 Configure the firmware

Edit `Configuration.h` and `Configuration_adv.h`. The fastest route is the config calculator: enter the worksheet values, then paste its two blocks over the matching lines. The lines below are what those blocks contain and why.

**Board and connection**, in `Configuration.h`:

```cpp
#define MOTHERBOARD BOARD_BTT_OCTOPUS_V1_1
#define SERIAL_PORT -1
#define EEPROM_SETTINGS
```

`EEPROM_SETTINGS` is load-bearing. Without it, `M500` reports that EEPROM is disabled, and every value tuned over USB is lost at the next power cycle.

**No printer hardware**, in `Configuration.h`:

```cpp
#define EXTRUDERS 0
#define TEMP_SENSOR_0 0
#define TEMP_SENSOR_BED 0
```

Setting `EXTRUDERS 0` removes the temperature subsystem entirely, so no thermistor is needed and no sensor value has to be faked. Older versions of these guides set `TEMP_SENSOR_0 998`, a dummy sensor reading a constant 25 °C. The dummy was necessary only because those guides left an extruder defined. With no extruder, plain `0` compiles and no dummy is required.

**Axis count.** Marlin sets `NUM_AXES` from the highest axis whose driver type is defined, so the declared axes always run contiguously from X. Two consequences follow, and both bite hard:

- Every axis array must carry exactly `NUM_AXES` entries. With `EXTRUDERS 0`, `LOGICAL_AXES` and `DISTINCT_AXES` both equal `NUM_AXES`, so one count governs every array in both files. An array of the wrong length stops the build with a static assertion naming the array.
- A stock configuration file defines Z and E0. Leaving them defined on a single-axis build makes Marlin count three axes, and every array sized for one is then rejected. Comment out the driver type for each axis not built.

**Declare two axes even for one drive unit.** At exactly one axis, Marlin 2.1.2.8 fails to compile `M211.cpp`: `print_pos` collapses to a single-float signature that collides with its vector overload, and the call is ambiguous. The fault is in Marlin, not in the configuration, and it is still present in the bugfix branch. Declaring a second axis avoids it. Declare a driver type for Y, fit no motor, and give it no endstop. Nothing homes it and nothing moves it.

```cpp
#define X_DRIVER_TYPE TMC2209
#define Y_DRIVER_TYPE TMC2209   // declared only, no motor fitted
//#define Z_DRIVER_TYPE A4988
//#define E0_DRIVER_TYPE A4988
```

> **Home with the axis named, as `G28 X`.** A bare `G28` homes every declared axis, including one with no switch to stop it.

**Motion**, in `Configuration.h`. Both arrays carry one entry per declared axis:

```cpp
#define DEFAULT_AXIS_STEPS_PER_UNIT { 320, 320 }
#define DEFAULT_MAX_FEEDRATE        { 120, 120 }
#define DEFAULT_MAX_ACCELERATION    { 3000, 3000 }
#define DEFAULT_ACCELERATION        500
#define DEFAULT_TRAVEL_ACCELERATION 500
```

`DEFAULT_MAX_FEEDRATE` and `DEFAULT_MAX_ACCELERATION` are ceilings that clamp what a command may reach. `DEFAULT_ACCELERATION` and `DEFAULT_TRAVEL_ACCELERATION` are the values moves actually use. Raising only the ceilings changes nothing observable.

**Bed size.** Marlin requires `X_BED_SIZE` and `Y_BED_SIZE` at any axis count, so both must appear even when no bed exists and no Y is built.

**Per-axis driver settings**, in `Configuration_adv.h`:

```cpp
#define X_CURRENT      1000
#define X_CURRENT_HOME 700
#define X_MICROSTEPS   16
#define X_RSENSE       0.11
#define X_CHAIN_POS    -1
```

A TMC5160 also needs `TMC_USE_SW_SPI` in the same file.

> **Confirm `X_RSENSE` against the physical driver module.** The sense resistor sets the scale for every commanded current, so a wrong value makes every current wrong. TMC2209 modules are commonly 0.11 Ω, a BTT TMC5160 is 0.075 Ω, and a BTT TMC5160T Plus is 0.022 Ω.

---

## 0.7 Calculate steps per mm

The drive unit ends in a 20-tooth GT2 pulley. GT2 has a 2 mm pitch, so one turn of that pulley moves the belt 40 mm.

Ahead of it sits a gearset: a 20-tooth pinion driving a larger gear. The reduction is the driven tooth count divided by 20, so the motor turns that many times for one turn of the output pulley.

```
steps_per_mm = full_steps × microsteps × (driven_teeth ÷ 20) ÷ 40
```

For a 1.8° motor at 16× microstepping this reduces to a single rule: **steps per mm equals four times the driven gear's tooth count.**

**Table 1. Steps per mm by gearset, 200-step motor, 20T output pulley.**

| Gearset | Reduction | 16× microstepping | 32× microstepping |
|---|---|---|---|
| 100T | 5:1 | 400 | 800 |
| 80T | 4:1 | 320 | 640 |
| 60T | 3:1 | 240 | 480 |
| Direct, 20T | 1:1 | 80 | 160 |

Every value is an exact integer, because 40 divides cleanly. That matters at the measurement in §0.9: a deviation of a few tenths of a percent points at mechanics, and a deviation of ten percent or more means the wrong gearset is entered in firmware. Verify which gearset is physically fitted rather than assuming, because entering a ratio from memory is the step at which the wrong number gets typed.

A 0.9° motor doubles every figure in Table 1.

---

## 0.8 Set the motor current

TMC drivers set current digitally, so no potentiometer needs turning. `X_CURRENT` is RMS current in milliamps.

The driver delivers peak current to one phase at a time, and RMS is peak divided by about 1.414. A starting value of 0.7 × the motor's rated phase current therefore keeps the peak at the motor's rating.

Then clamp that figure to what the driver sustains, and use the lower of the two.

**Table 2. Driver current ceilings.**

| Driver module | Sense resistor | Sustained RMS current |
|---|---|---|
| TMC2209 | 0.11 Ω | 1400 mA |
| BTT TMC5160 / 5160T Pro | 0.075 Ω | about 3100 mA |
| BTT TMC5160T Plus | 0.022 Ω | confirm from the product page |

The TMC2209 figure needs explaining, because its datasheet quotes 2 A RMS. That 2 A is the silicon maximum. The same datasheet warns that thermal design becomes critical at or above 1.4 A RMS for this package, and a module driven near 2 A reaches thermal shutdown within minutes. Size to the sustained figure, not the maximum.

A motor whose rating exceeds what the driver sustains is simply mismatched to it. Pairing a 2.8 A motor with a TMC2209 leaves a driver that can never reach the motor's rating, and a config calculator that computes 0.7 × 2.8 A gives 1950 mA, which the driver cannot hold.

Change current over USB without reflashing:

```
M906 X1000
M906
M500
```

---

## 0.9 Build, flash and first motion

1. Build with the Auto Build Marlin button. The environment is `STM32F446ZE_btt` for an F446 board, or `STM32F429ZG_btt` for an F429.
2. Copy `.pio/build/<env>/firmware.bin` to the FAT32 card.
3. Insert the card and power-cycle the board.
4. Confirm the bootloader renamed the file to `firmware.CUR`. If it did not, the flash failed; check that the name is exactly `firmware.bin` and the card is FAT32.
5. Open a serial terminal over USB at 250000 baud.

**Confirm the driver is talking.** Send `M122`. A healthy driver returns real register values. All zeros or all ones means the board is not reaching the driver. Recheck the mode jumpers, the driver seating, and the driver type in firmware.

**Confirm which firmware is running.** Send `M115` and read the build timestamp. A stale timestamp means the flash did not take, and every test after that measures the old firmware. Check this before interpreting any other result.

**Jog the motor:**

```
G91
G1 X10 F600
G1 X-10 F600
G90
```

If the motor turns the wrong way, set `INVERT_X_DIR true` and reflash, or swap one coil's two wires.

**Verify steps per mm.**

1. Mark the carriage position.
2. Command a known distance.
3. Measure the actual travel with calipers.
4. Compute the true value:

```
true_steps_per_mm = configured_value × (commanded ÷ measured)
```

Compare the result against Table 1. A result within a fraction of a percent of the configured value means the drivetrain is sound. A result close to a *different* row means the wrong gearset is configured. No amount of belt tensioning corrects that. Apply the corrected value and save:

```
M92 X320
M500
```

Then write the same value into `DEFAULT_AXIS_STEPS_PER_UNIT`, so it survives an EEPROM reset.

> **After changing a default and reflashing, send `M502` then `M500`.** A value stored by `M500` overrides the firmware default at every boot. Skip this and the old value wins silently, so the new build appears to have changed nothing.

---

## 0.10 Troubleshooting

| Symptom | Cause and fix |
|---|---|
| Motor silent, no holding torque, `M122` all zeros or all ones | Driver mode jumpers, seating, or driver type. Power off and recheck §0.3 and §0.4. |
| Motor buzzes, locks or twitches but does not turn | Coil pairing wrong. Power off and re-pair (§0.4). |
| Distance wrong by a few tenths of a percent | Mechanical. Check belt tension and pulley grub screws. |
| Distance wrong by ten percent or more | Wrong gearset configured. Compare the implied value against Table 1. |
| Motor skips under load | Current too low, or commanded speed above what the motor can hold. |
| Motor or driver runs hot | Current too high. Drop a level and add airflow. Confirm `X_RSENSE`. |
| Build stops on a static assertion naming an array | Array length does not match `NUM_AXES` (§0.6). |
| Build stops in `M211.cpp` on an ambiguous call | Exactly one axis declared. Declare a second (§0.6). |
| No serial port | Charge-only cable, wrong port, or `SERIAL_PORT` is not `-1`. |
| `M500` reports EEPROM disabled | `EEPROM_SETTINGS` is commented out (§0.6). |
| New firmware behaves like the old one | Check the `M115` build timestamp, then clear stale EEPROM with `M502` and `M500`. |

---

## 0.11 The Octopus Max EZ

The Max EZ uses an STM32H723 and pinless driver sockets, so it needs no mode jumpers. Use `BOARD_BTT_OCTOPUS_MAX_EZ` and the `STM32H723Zx_btt` environment. The motion configuration is unchanged. Confirm pin names against its own pins file.

---

## Appendix A — A verified single-axis configuration

Every value below was built, flashed and measured on real hardware. Treat it as a reference point rather than a template: the gearset, motor and travel of another build will differ.

**The machine.** One drive unit on an Octopus V1.1, MOTOR0, with the endstop on the X-STOP header (PG6). An 80T gearset, a 1.8° motor at 16× microstepping, and a TMC2209. A second axis is declared with no motor fitted, to reach two declared axes.

**Key values.**

| Setting | Value | Where it came from |
|---|---|---|
| `DEFAULT_AXIS_STEPS_PER_UNIT` | `{ 320, 320 }` | 4 × 80T, then confirmed by measurement |
| `DEFAULT_MAX_FEEDRATE` | `{ 334, 334 }` | measured stability ceiling, see below |
| `DEFAULT_MAX_ACCELERATION` | `{ 20000, 20000 }` | ceiling only, not the operating value |
| `DEFAULT_ACCELERATION` | `3000` | the value moves use |
| `X_CURRENT` | `1400` | 0.7 × rated, clamped to the TMC2209 sustained figure |
| `X_CURRENT_HOME` | `1000` | reduced, so homing meets the stop gently |

**The measurement that set steps per mm.** Configured at 400, a commanded 10 mm produced 12.52 mm of travel. That gives 4000 ÷ 12.52 = 319.5 steps per mm, which is 0.16 % from 320 and identifies the 80T gearset. The configured 400 had come from assuming a 100T gearset.

**The measurement that set the feedrate ceiling.** Above F20000, the axis skipped steps. Below it, position held exactly across hundreds of moves. F20000 is 333.33 mm/s, so the ceiling is set to 334 rather than 333: a ceiling of 333 gives `M203 X333.00`, which is 19980 mm/min, and rejects the very feedrate that was proven good.

**What skipping looked like.** Above the ceiling, the firmware kept counting steps the motor did not take, so the carriage fell behind its commanded position. After enough cycles it reached the physical switch while the firmware still believed it was 50 mm away. The switch stopped it; the software limits could not, because they are computed from the commanded position. Open-loop control means exactly that gap between commanded and actual. A speed test must therefore re-home and re-check position after every rung, rather than relying on how the axis sounds.

---

**Next:** with one unit moving a verified distance, Guide 1 adds a home reference.
