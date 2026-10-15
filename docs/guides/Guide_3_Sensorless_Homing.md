# Guide 3 — Sensorless homing

**Scope.** Home an axis with no limit switch. The driver senses the rise in motor load as the carriage reaches the end of travel and reports it as a stall, which Marlin uses in place of a switch.

This guide replaces the sensorless material that previously sat inside Guide 1 and Guide 2. Set an axis up here, or with a switch in Guide 1, but not both at once on the same axis. The Advanced guide covers carrying both.

**Firmware target.** Marlin 2.1.2.8, Octopus V1.1 pins.

**Drivers.** TMC2209 uses StallGuard4, TMC5160 uses StallGuard2. The two use different sensitivity scales, running in opposite directions, and §3.3 is the section that keeps that from costing an afternoon.

---

## 3.1 Why a switch and sensorless cannot share an axis

Enabling sensorless homing makes Marlin route the driver's stall output onto that axis's normal endstop pin. For X on the Octopus, that pin is PG6, the X-STOP header.

A physical switch wired to X-STOP would then contend with the stall signal on the same pin. The two are mutually exclusive on a given axis unless the switch is moved to a spare input, which is what the Advanced guide does.

---

## 3.2 Fit the jumper and configure

### 3.2.1 Fit the DIAG jumper

1. Disconnect all power.
2. Fit the DIAG jumper for the axis's slot.
3. Confirm no physical switch remains wired to that axis's endstop header.

The jumper connects the driver's diagnostic output to the endstop input.

### 3.2.2 Configure

In `Configuration_adv.h`:

```cpp
#define SENSORLESS_HOMING

#if EITHER(SENSORLESS_HOMING, SENSORLESS_PROBING)
  #define X_STALL_SENSITIVITY 100   // see §3.3 — the valid range depends on the driver fitted
  #define IMPROVE_HOMING_RELIABILITY
#endif
```

A stall cannot be detected twice in a row at different speeds, so disable the slow second bump and back the carriage off afterwards:

```cpp
#define X_HOME_BUMP_MM 0
```

Reduce the current used while homing, so the carriage meets the end gently and repeatably:

```cpp
#define X_CURRENT_HOME 700    // roughly 70 % of the run current
```

> **Set the run current before tuning sensitivity.** StallGuard reads motor load, which scales with current, so a sensitivity tuned at one current is wrong at another.

---

## 3.3 The sensitivity scale differs by driver, and so does its direction

This is the most common source of wasted time when moving between the two drivers.

**Table 1. Stall sensitivity by driver.**

| Driver | Feature | Valid range | Direction |
|---|---|---|---|
| TMC2209 | StallGuard4 | 0 to 255 | higher is more sensitive |
| TMC5160 | StallGuard2 | −64 to +63 | lower is more sensitive |

The ranges do not overlap in meaning, and the directions are opposite. A value of 80 is a reasonable mid-scale starting point on a TMC2209. The same 80 is **outside the valid range** on a TMC5160, where the scale stops at +63.

Confirm which driver the axis carries before choosing a starting value. A sensible start is around 100 on a TMC2209, and around 0 on a TMC5160.

---

## 3.4 Tune the sensitivity

Tune live over USB, without reflashing between attempts.

1. Build, flash and connect.
2. Send `M115` and confirm the build timestamp.
3. Set a starting sensitivity with `M914 X100`, on the scale for the driver fitted.
4. Send `G28 X`.
5. Observe where the axis stops.
6. Adjust and repeat.

Read the result as follows:

- **Stops early, before reaching the end.** Too sensitive. Move toward the less sensitive end of the scale.
- **Grinds at the end without stopping.** Not sensitive enough. Move toward the more sensitive end.
- **Stops at the end, repeatably, with no false triggers.** Correct.

Repeat the homing move several times at the chosen value before accepting it. A value that works once and fails on the third attempt is not tuned.

Then write the value into `X_STALL_SENSITIVITY`, reflash, and send `M500`.

---

## 3.5 Homing speed changes the reading

StallGuard infers load from back-EMF, which depends on motor speed. Too slow and the signal is too weak to read; too fast and the reading becomes unreliable.

Keep the homing feedrate moderate, and **re-tune the sensitivity whenever the homing speed or the run current changes.** A gearset lowers axis speed for a given motor speed, which generally helps, but confirm the motor still turns fast enough at the chosen homing feedrate to give a clean reading.

---

## 3.6 Where sensorless homing is the wrong choice

**A vertical axis.** Gravity loads a vertical axis differently in each direction, and the carriage's own weight is indistinguishable from the stall the driver is watching for. Use a switch or a probe on a vertical Z.

**An axis that must not touch its end stop.** Sensorless homing works by driving into the end and detecting the collision. Where the mechanism cannot tolerate that contact, fit a switch.

**An axis carrying a variable load.** The stall threshold that suits an empty carriage will trigger early under load. A switch reads the same regardless.

---

## 3.7 Switching between a switch and sensorless later

Changing method means both rewiring and reconfiguring:

1. Disconnect all power.
2. Fit or remove the DIAG jumper.
3. Fit or remove the physical switch.
4. Enable or disable `SENSORLESS_HOMING`.
5. Rebuild and reflash.

The wiring and the firmware must agree. A DIAG jumper fitted with sensorless disabled, or a switch wired to the endstop pin with sensorless enabled, gives unpredictable behaviour on that pin.

To avoid rewiring on every change, build the arrangement in the Advanced guide, which keeps both physically present on separate inputs.

---

**Next:** the Advanced guide carries a switch and sensorless homing on one axis at the same time.
