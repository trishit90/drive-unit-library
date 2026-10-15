# Guide 2 — Composing drive units into a multi-axis machine

**Scope.** Take several drive units, each moving a verified distance (Guide 0) and homing reliably on its own (Guide 1), and build the firmware for a machine that uses them together. Worked here as X, Y, Z, a second Z motor for gantry squaring, and a rotary axis.

**Premise.** Validate every unit on its own first. Debugging a four-axis machine while unsure whether any single axis works costs far more than confirming each one in isolation.

**Firmware target.** Marlin 2.1.2.8, Octopus V1.1 pins.

---

## 2.1 Driver slot budget

The Octopus has eight driver slots.

**Table 1. Default slot to axis mapping.**

| Slot | Marlin axis | Typical use |
|---|---|---|
| MOTOR0 | X | X axis |
| MOTOR1 | Y | Y axis |
| MOTOR2 | Z | Z axis |
| MOTOR3 | Z2 | second Z motor |
| MOTOR4 | E0 | extruder, or repurposed as a rotary axis |
| MOTOR5 to MOTOR7 | E1 to E3 | spare |

Record which physical slot carries each logical axis. That note is the detail most likely to trip up a rebuild months later.

---

## 2.2 Get the axis count right before anything else

Marlin sets `NUM_AXES` from the highest axis whose driver type is defined, and the declared axes always run contiguously from X. Guide 0 §0.6 covers the mechanism; the consequences for a multi-axis build are these:

- Declaring a rotary axis as I makes `NUM_AXES` four, which means Y and Z are declared whether or not they were built.
- Every axis array in both configuration files must carry exactly `NUM_AXES` entries. With `EXTRUDERS 0`, that single count governs all of them.
- Comment out the driver type for every axis not built, including the `E0_DRIVER_TYPE` a stock file defines.

The arrays governed by the count are `DEFAULT_AXIS_STEPS_PER_UNIT`, `DEFAULT_MAX_FEEDRATE`, `DEFAULT_MAX_ACCELERATION` and `HOMING_FEEDRATE_MM_M` in `Configuration.h`, and `HOMING_BUMP_MM`, `HOMING_BUMP_DIVISOR`, `HOMING_BACKOFF_POST_MM` and `AXIS_RELATIVE_MODES` in `Configuration_adv.h`.

> **Re-check every axis array before enabling a display or sensorless homing.** `MANUAL_FEEDRATE`, `SENSORLESS_BACKOFF_MM` and the `*_EDIT_VALUES` arrays sit behind disabled features at their stock length. Enabling any of those features on a reduced-axis machine produces a fresh wave of length errors.

---

## 2.3 Replicate the per-axis configuration

Each axis takes the same block as X, with the letter changed. Driver type, current, microstepping, sense resistor and steps per mm are all per axis and may differ. A heavily loaded Z can carry a different gearset from X.

```cpp
#define X_DRIVER_TYPE TMC2209
#define Y_DRIVER_TYPE TMC2209
#define Z_DRIVER_TYPE TMC5160
```

Steps per mm follows each axis's own gearset, from Guide 0 Table 1. A machine mixing an 80T gearset on X and Y with a 60T on Z gives:

```cpp
#define DEFAULT_AXIS_STEPS_PER_UNIT { 320, 320, 240 }
```

Set current per axis from that axis's motor and driver, clamped to the driver's sustained rating (Guide 0 §0.8). Any TMC5160 in the machine also needs `TMC_USE_SW_SPI`, which covers every SPI driver on the shared bus.

---

## 2.4 Per-axis endstops and homing order

Set each axis's homing direction and endstop as Guide 1 describes, and measure each axis's polarity separately.

```cpp
#define X_HOME_DIR -1
#define Y_HOME_DIR -1
#define Z_HOME_DIR -1
```

Enable each axis's plug, and enable `ENDSTOPS_ALWAYS_ON_DEFAULT` once for the machine.

Mixed strategies across axes are fine. Sensorless homing on X and Y with a physical switch on Z is a common and sound arrangement, because sensorless homing on a vertical axis is unreliable (Guide 3).

Control the homing order so the machine does not collide with itself:

```cpp
#define Z_SAFE_HOMING     // when homing Z needs X and Y at a known position first
```

---

## 2.5 A gantry with two motors on one axis

A wide gantry driven by two motors can rack: one side leads the other, and the gantry sits out of square. Two motors with two switches let the firmware square it, by driving each motor to its own switch independently.

This section discharges the forward reference in Guide 1 §B.3.

### 2.5.1 Wire the second switch

1. Disconnect all power.
2. Fit the second motor to MOTOR3.
3. Wire its switch to a header that is not already carrying an endstop.
4. Confirm the header against the board's pinout, and note which pin it is.

The stock mapping for the second Z switch lands on an E-detector header. Many builds instead use the Z-MAX header, which needs the pin stated explicitly:

```cpp
#define USE_ZMAX_PLUG
#define Z_MAX_PIN PG11    // second Z switch on the Z-MAX header; confirm against the board pinout
#define Z_MAX_ENDSTOP_INVERTING true   // measured, as every other endstop
```

### 2.5.2 Configure the squaring

In `Configuration_adv.h`:

```cpp
#define Z_MULTI_ENDSTOPS          // each Z motor has its own switch
#define Z2_USE_ENDSTOP _ZMAX_     // which plug the second switch uses
```

The second motor mirrors the first for everything except its switch. Marlin's own idiom is to alias rather than repeat the numbers, so a later change to Z carries across:

```cpp
#define Z2_CURRENT      Z_CURRENT
#define Z2_CURRENT_HOME Z_CURRENT_HOME
#define Z2_MICROSTEPS   Z_MICROSTEPS
#define Z2_RSENSE       Z_RSENSE
#define Z2_CHAIN_POS    -1
```

### 2.5.3 Prove the squaring

1. Send `M119` and confirm both Z switches read independently.
2. Press one switch and confirm only its own state changes.
3. Home Z with the gantry deliberately racked by a few millimetres.
4. Confirm both sides meet their switches and the gantry ends square.

> **Set both switches at the same height before squaring.** The firmware squares the gantry to the switches, so a switch mounted crooked squares the gantry crooked.

---

## 2.6 A rotary axis

A rotary axis turns rather than traverses, so it is configured in steps per degree. Marlin's additional axes are I, J and K, and a rotary axis normally maps to I.

```cpp
#define I_DRIVER_TYPE TMC2209
#define AXIS4_NAME 'A'
#define AXIS4_ROTATES
#define I_ENABLE_ON 0     // required for any axis beyond Z
```

`I_ENABLE_ON` is easy to miss. Without it the build stops with a message naming the I stepper.

Steps per degree follows the same drivetrain arithmetic with 360 in place of the 40 mm:

```
steps_per_deg = full_steps × microsteps × (driven_teeth ÷ 20) ÷ 360
```

For a 200-step motor at 16× with an 80T gearset, that is 12800 ÷ 360 = 35.556 steps per degree.

Unlike the linear figures, these are not integers, so rounding accumulates over many turns. That is acceptable for most rotary work. Where exact indexing matters, choose a combination dividing 360 cleanly, then confirm by commanding a full turn and measuring the actual angle.

### 2.6.1 Repurposing an extruder slot

Putting the rotary axis on MOTOR4 means it uses the first extruder slot's pins. Marlin's board file maps no I axis, so every pin must be stated:

```cpp
#define I_STEP_PIN      PF9
#define I_DIR_PIN       PF10
#define I_ENABLE_PIN    PG2
#define I_SERIAL_TX_PIN PF2
#define I_SERIAL_RX_PIN PF2
#define I_MIN_PIN       PG12
```

Moving the axis to a different slot means editing all six lines.

A rotary axis often has no endstop. Where it needs an index reference, fit a switch and home it like a linear axis. Otherwise leave it unhomed and set its position with `G92`.

---

## 2.7 Self-holding axes

The gearset makes each axis resist back-driving when unpowered, so a carriage holds position with the motor de-energised. A vertical Z will not sag when the steppers are released.

Confirm it per axis rather than assuming it. Disable the motor, push the carriage by hand under the load it will carry, and check that it does not creep.

The same property removes any means of moving a carriage clear by hand, which matters during first homing.

---

## 2.8 Bring-up order for the assembled machine

Work one axis at a time. A wrong direction or a wrong polarity is far easier to attribute before several axes move together.

1. Flash the combined configuration.
2. Send `M115` and confirm the build timestamp matches the firmware just flashed.
3. Send `M122` and confirm every driver reports real register values.
4. Send `M119` and confirm every endstop reads correctly, open when free and triggered when pressed.
5. Jog each axis a small known distance with `G91` and a short `G1`, confirming direction and distance.
6. Home each axis on its own, naming it, as `G28 X` then `G28 Y` then `G28 Z`.
7. Run a full `G28` only after every axis homes correctly alone.
8. Send `M500` to persist.

> **Do not run a bare `G28` while any declared axis lacks a switch.** A bare `G28` homes every declared axis, and an axis with no switch to stop it runs its full travel and fails.

---

**Next:** Guide 3 covers sensorless homing, which replaces the switch entirely. The Advanced guide covers carrying both on one axis.
