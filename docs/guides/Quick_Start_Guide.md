# Quick start — first motion and a measured baseline

**Purpose.** Configure and flash the firmware, home the axis, and measure what the unit can do under its working load.

**Result.** A working homed axis, and four measured numbers: steps per mm, top speed, acceleration and motor current.

**Time.** About two hours.

**Versions.** Marlin 2.1.2.8 on a BTT Octopus V1.1. Do not mix this guide with instructions written for another version.

---

## Prerequisites

Confirm every line. Where one fails, the guide named covers it.

- [ ] Drive unit assembled, belt tensioned
- [ ] One limit switch mounted at the home end and wired to the X-STOP header (Guide 1 §A.1)
- [ ] Driver seated in the X slot, mode jumpers set for UART or SPI (Guide 0 §0.3)
- [ ] Motor wired, coil pairs confirmed (Guide 0 §0.4)
- [ ] 24 V supply wired, polarity checked (Guide 0 §0.4)
- [ ] VS Code and Auto Build Marlin installed, Marlin 2.1.2.8 downloaded (Guide 0 §0.5)
- [ ] microSD card formatted FAT32, and a data-capable USB-C cable

---

## Step 1 — Fill in the worksheet

Six values. Everything else follows from them.

**Table 1. Worksheet.**

| # | Parameter | Where to find it | Measured value |
|---|---|---|---|
| P1 | Usable travel, mm | Turn the pulley by hand to move the carriage. Measure from where the switch clicks to the far hard stop, then subtract 5 mm. | |
| P2 | Motor rated current per phase, A | Printed on the motor label or its datasheet. | |
| P3 | Motor full steps per revolution | From the step angle: 1.8° is 200, and 0.9° is 400. | |
| P4 | Driver module in the X slot | TMC2209, BTT TMC5160 or 5160T Pro, or BTT TMC5160T Plus. The exact module sets the sense resistor. | |
| P5 | Gearset fitted | Read the tooth count off the driven gear: 100T, 80T, 60T, or direct drive. | |
| P6 | Microstepping | Use 16. | |

> **Read P5 off the physical gear rather than recalling it.** Entering the wrong gearset is the single most common cause of an axis that moves the wrong distance, and the error looks exactly like a mechanical fault.

---

## Step 2 — Generate the configuration

Open **drive_unit_calculator.html**, enter P1 to P6, and check the axis map it shows. For one drive unit it will report two declared axes, because Marlin cannot compile a single-axis build (Guide 0 §0.6).

The calculator produces two blocks of configuration, one per file. It also computes:

- **Steps per mm**, which is four times the gearset tooth count at 16× microstepping.
- **Run current**, which is 0.7 × P2, clamped to what the driver sustains.

Check the derived values against the expected figures before going on. A steps-per-mm figure that does not match the gearset fitted means the wrong gearset is selected.

Then click **Download profile (JSON)**. The operator panel imports that file and cross-checks the predicted steps per mm against what the board reports, at every connect.

---

## Step 3 — Apply the configuration

1. Open the Marlin project folder in VS Code.
2. Open `Marlin/Configuration.h`.
3. For each line in the calculator's first block, find the same define in the file and replace it.
4. Save the file.
5. Open `Marlin/Configuration_adv.h`.
6. Repeat for the calculator's second block.
7. Save the file.

Two points about that replacement, both of which cost time when missed:

**Some lines are already correct.** A stock Marlin file already holds many of these values. Replacing a line with what it already said is not an error and not a wasted step; it confirms the value rather than changing it. Do not skip a line because it looks unchanged.

**Some lines must be uncommented, and some must be commented out.** The calculator emits both. A line it shows as `//#define Z_DRIVER_TYPE A4988` must end up commented out, because leaving it defined makes Marlin count an axis that was never built and rejects every array (Guide 0 §0.6).

✓ **Result:** both files saved, with every define from the two blocks present and matching.

---

## Step 4 — Build and flash

1. Click the Auto Build Marlin **Build** button. The environment is `STM32F446ZE_btt` on the common board.

   ✓ **Result:** the log ends in SUCCESS and `firmware.bin` exists under `.pio/build/`.

2. Copy `firmware.bin` to the microSD card.
3. Insert the card and power-cycle the board.

   ✓ **Result:** after about ten seconds the file on the card is renamed `firmware.CUR`. If it is not, check the name is exactly `firmware.bin` and the card is FAT32.

---

## Step 5 — Connect

Use the operator panel, or any host that speaks G-code over USB at 250000 baud.

1. Plug in the USB-C cable.
2. Connect.
3. Send `M115`.

   ✓ **Result:** the reported build timestamp matches the firmware just flashed.

> **Check that timestamp before interpreting any later result.** A stale timestamp means the flash did not take, and every test after it measures the old firmware.

4. Send `M503` and confirm the steps per mm, current and feedrates match what the calculator produced.

If `M503` shows values that were not set here, a previous `M500` stored them and they override the firmware defaults. Send `M502` then `M500` to clear the stored copy, then power-cycle and check again.

---

## Step 6 — Prove the switch before any motion

1. Send `M119` with the carriage clear of the switch.

   ✓ **Result:** `x_min: open`

2. Hold the switch closed and send `M119` again.

   ✓ **Result:** `x_min: TRIGGERED`

> **Do not home until both readings are correct.** A switch reading the wrong way round cannot stop the carriage. Where the readings are swapped, flip `X_MIN_ENDSTOP_INVERTING` and reflash.

3. Send `M120` to enable the endstops for ordinary moves as well as homing.

Marlin reads the endstops only while homing unless told otherwise. `ENDSTOPS_ALWAYS_ON_DEFAULT` makes that permanent, and `M120` proves the behaviour before committing to a reflash (Guide 1 §A.3).

---

## Step 7 — Home for the first time

> **Rest a finger near the switch before sending the command.** The direction is unproven, and the geared axis is self-holding, so the switch is the only stop.

1. Send `G28 X`.
2. Watch the first centimetre of travel.
3. If the carriage moves toward the switch, let it finish.

   ✓ **Result:** it touches, backs off, re-touches slowly, and rests a short distance clear.

4. If the carriage moves away from the switch, close the switch by hand to stop it. Set `INVERT_X_DIR true`, redo Step 4, and repeat.

---

## Step 8 — Prove the distance

1. Send `G90` then `G1 X50 F1200`. Use a shorter distance if P1 is under 60 mm.
2. Measure the actual travel with a rule or calipers.

   ✓ **Result:** the travel matches the command within about 0.5 mm.

Where it does not match, compute the true value and compare it against the gearset table in Guide 0 §0.7:

```
true_steps_per_mm = configured × (commanded ÷ measured)
```

A result within a fraction of a percent of the configured value is mechanical: check belt tension and pulley grub screws. A result landing on a *different* gearset row means the wrong gearset is configured, and tensioning the belt will not fix it. Apply the corrected value with `M92 X⟨value⟩` and `M500`, then put it into Step 2's input so the next flash carries it.

---

## Step 9 — Set up the baseline test

> **Units.** `F` in a G-code command is millimetres per minute. Configuration values are millimetres per second. Multiply by 60 to convert.

1. Fit the real load the unit will carry. The numbers are valid only for the load tested.
2. Send `G28 X` then `G1 X0`.
3. Fix a tape flag on the rail, aligned with an edge of the carriage. A dial indicator is better where one is available.
4. Choose a test stroke **d** from Table 2, at least as long as the highest rung to be tested and no more than P1 − 10.
5. Send `M201 X3000` then `M204 T500`.

> **Plain moves obey `M204 T`, not `M204 P`.** Setting P on a motion platform changes nothing, which is the most common silent tuning error.

---

## Step 10 — Test A, top speed

For each rung in order:

1. Send `G28 X` then `G1 X0`, and confirm the carriage lines up with the mark.
2. Send the out-and-back pair three times at the rung's F value.
3. Check the mark.
4. Record the result, and stop at the first failure.

A rung passes when the carriage realigns with the mark and nothing grinds or judders. It fails on any misalignment, or any grinding or judder during a move.

**Table 2. Speed ladder.**

| Rung | Speed, mm/s | F value | Stroke needed, mm |
|---|---|---|---|
| 1 | 10 | F600 | 1 |
| 2 | 20 | F1200 | 2 |
| 3 | 30 | F1800 | 4 |
| 4 | 40 | F2400 | 7 |
| 5 | 50 | F3000 | 10 |
| 6 | 75 | F4500 | 23 |
| 7 | 100 | F6000 | 40 |

The stroke column is the distance the axis needs to reach that speed and slow down again, at 500 mm/s². A shorter stroke never reaches the speed, so the rung tests nothing.

> **Re-home between rungs and check the mark, rather than judging by sound.** Once an axis skips steps the firmware's position no longer matches the machine, and the software limits are then protecting a position that has drifted. The mark is an absolute physical reference; the position report is not.

✓ **Result:** speed ceiling is the last rung that passed. Working maximum is 0.8 × the ceiling.

**Converting the ceiling into a firmware limit.** Set `DEFAULT_MAX_FEEDRATE` to the ceiling in mm/s, rounding **up** to the next whole number. Rounding down puts the limit below the feedrate just proven, and the firmware then rejects it. A ceiling of F20000 is 333.33 mm/s, so the limit is 334, not 333.

---

## Step 11 — Test B, acceleration

1. Fix the test speed at 70 % of the Test A ceiling.
2. For each rung, send `M201 X⟨rung⟩` then `M204 T⟨rung⟩`.
3. Run the same three out-and-back moves and check the mark.

**Table 3. Acceleration ladder.**

| Rung | Acceleration, mm/s² |
|---|---|
| 1 | 300 |
| 2 | 500 |
| 3 | 800 |
| 4 | 1200 |
| 5 | 1800 |
| 6 | 2500 |

✓ **Result:** acceleration ceiling is the last rung that passed. Working value is 0.8 × the ceiling.

---

## Step 12 — Test C, current

For each level, compute the milliamp figure as the multiplier × P2 × 1000, rounded to the nearest 50, then clamped to the driver's sustained rating (Guide 0 Table 2).

1. Send `M906 X⟨mA⟩`.
2. Re-run the Test A ceiling rung three times at the Test B working acceleration.
3. Check the mark.
4. Send `M122` and record any `otpw` or `ot` flag. `otpw` is an over-temperature pre-warning, and `ot` is a shutdown.
5. Touch the motor casing briefly. Uncomfortably hot means back off a level.

**Table 4. Current levels.**

| Level | Multiplier of rated |
|---|---|
| 1 | 0.6 |
| 2 | 0.7 |
| 3 | 0.8 |

✓ **Result:** chosen current is the lowest level that holds the ceiling with no `otpw`.

---

## Step 13 — Save the result

1. Send the working values and persist them:

```
M203 X⟨working max speed, mm/s⟩
M201 X⟨working acceleration⟩
M204 T⟨working acceleration⟩
M906 X⟨chosen mA⟩
M500
M503
```

   ✓ **Result:** `M503` echoes the values set.

2. Put the same numbers into the source, so they survive an EEPROM reset: `DEFAULT_MAX_FEEDRATE`, `DEFAULT_MAX_ACCELERATION`, `DEFAULT_ACCELERATION` and `DEFAULT_TRAVEL_ACCELERATION` in `Configuration.h`, and `X_CURRENT` in `Configuration_adv.h`.

3. Write the baseline record:

> Unit ⟨id⟩ · load ⟨description⟩ · ⟨gearset⟩ at 16× · working maximum ⟨speed⟩ mm/s at ⟨acceleration⟩ mm/s², ⟨current⟩ mA RMS · ⟨date⟩

---

## If something goes wrong

**Table 5. Symptoms and causes.**

| Symptom | Cause and fix |
|---|---|
| No serial port appears | Charge-only cable, wrong port, or `SERIAL_PORT` is not `-1` |
| Build stops on a static assertion naming an array | An axis array does not match the axis count (Guide 0 §0.6) |
| Build stops in `M211.cpp` on an ambiguous call | Only one axis declared; declare a second (Guide 0 §0.6) |
| `M119` reads inverted | Flip `X_MIN_ENDSTOP_INVERTING` and reflash |
| First home drives away from the switch | Close the switch by hand, then set `INVERT_X_DIR true` |
| `M122` shows all zeros or all FFs | Driver mode jumpers, seating, or driver type (Guide 0 §0.3) |
| Distance wrong by ten percent or more | Wrong gearset configured (Step 8) |
| Distance wrong by under one percent | Belt tension or pulley grub screws |
| `M906` behaves strangely | `X_RSENSE` does not match the driver module |
| A rung fails only at high speed on a short stroke | The axis never reached the speed; use the stroke column |
| Switch trips but the axis keeps moving | Endstops are only read while homing; send `M120` (Step 6) |
| `M114` reports a position the carriage never reached | Expected after an endstop stop outside `G28`; re-home |
| New firmware behaves like the old | Check the `M115` timestamp, then clear stale EEPROM with `M502` and `M500` |
| `M500` reports EEPROM disabled | `EEPROM_SETTINGS` is commented out |

---

## Questions this raises

**Do I need to wire a thermistor or a heater?**
No. With `EXTRUDERS 0` there is no temperature subsystem at all, so nothing needs wiring and no sensor value needs faking. Earlier versions of these guides used a dummy sensor reading 25 °C; that was only necessary because they left an extruder defined.

**Why must a single drive unit declare two axes?**
Marlin 2.1.2.8 cannot compile a one-axis build. `print_pos` collapses to a signature that collides with another overload, and `M211.cpp` fails on an ambiguous call. Declaring a second axis with no motor avoids the fault and keeps the software limits available.

**Why is the top speed low compared with a 3D printer?**
The gearset multiplies motor turns per millimetre of carriage travel, and stepper torque falls steeply past the motor's knee speed. An 80T gearset gives 10 mm of travel per motor turn, so 200 mm/s is already 1200 rpm at the motor. That trade buys holding torque and resolution.

**Why a tape mark instead of reading the position back?**
`G28` resets the position counter, so after re-homing the reported position cannot reveal a loss. The mark is an absolute physical reference, and a real skip loses a visible millimetre or more.

**Why is the switch wired normally closed?**
A broken wire then reads as triggered and stops motion, rather than failing to stop it.

**Why 0.7 × the motor's rated current?**
Marlin's values are RMS, and the driver applies peak, which is about 1.414 × RMS, to one phase at a time. The 0.7 factor keeps that peak at the motor's rating. Clamp the result to the driver's sustained rating, which for a TMC2209 is well below its quoted maximum.

**Do `G0` and `G1` differ?**
Not in this build. Marlin separates them only when `G0_FEEDRATE` is defined, which it is not here, so both route through the same handler.

---

*Where this sits: Quick start (this guide) ↔ Guide 0 (full bench bring-up) → Guide 1 (limit switches) → Guide 2 (multi-axis) → Guide 3 (sensorless homing) → Advanced (both on one axis).*
