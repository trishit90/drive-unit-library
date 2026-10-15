# Guide 1 — Adding a home reference with limit switches

**Scope.** Take one drive unit that already moves a verified distance (Guide 0) and give it a home reference, so the firmware knows where the carriage is. This guide covers physical limit switches. Sensorless homing, which needs no switch, has its own guide (Guide 3) because it shares an input pin with the switch and cannot be combined with it on the same axis.

**Firmware target.** Marlin 2.1.2.8, Octopus V1.1 pins.

**Two variants.** Choose one:

- **Variant A, one switch** at the home end. The simplest and most robust starting point.
- **Variant B, two switches**, one at each end, adding a hard limit at the far end.

---

## 1.1 Decide the homing direction first

Homing drives the carriage toward one end until the switch triggers, then calls that point the origin. For a single axis treated as X, homing to the minimum end is conventional.

```cpp
#define X_HOME_DIR -1     // -1 homes toward minimum, +1 toward maximum
```

Set the travel so the software limits mean something:

```cpp
#define X_BED_SIZE 200    // usable travel in mm; X_MAX_POS follows it
#define X_MIN_POS 0
```

`X_BED_SIZE` is Marlin's name for the travel, and it carries that name whether or not a bed exists.

---

## Variant A — One limit switch

### A.1 Wire the switch

The X-STOP header carries signal, ground and 3.3 V. Wire the switch between signal and ground.

1. Disconnect all power.
2. Wire the switch common to ground.
3. Wire the switch normally-closed contact to signal.
4. Confirm the header's signal and ground pins against the board's pinout.

A normally-closed switch is preferred because a broken wire then reads as triggered and stops motion, rather than failing to stop it.

### A.2 Configure

```cpp
#define USE_XMIN_PLUG
#define ENDSTOPPULLUPS
#define X_HOME_DIR -1
#define X_MIN_ENDSTOP_INVERTING true
```

**Set the inverting value from a measurement, not from the switch type.** Marlin marks an endstop triggered when the pin reads the opposite of `*_ENDSTOP_INVERTING`. The rule is short:

| Voltage at the pin when triggered | Set `*_ENDSTOP_INVERTING` to |
|---|---|
| LOW | `true` |
| HIGH | `false` |

Wiring, switch type and any endstop module between the two all change which of these applies, so measure the pin rather than reasoning from the parts. Polarity is per axis: a machine can be triggered-low on one axis and triggered-high on another.

### A.3 Turn the endstops on for normal moves

By default Marlin reads the endstops **only while homing** and ignores them during every other move. A switch that trips mid-move then stops nothing.

In `Configuration_adv.h`:

```cpp
#define ENDSTOPS_ALWAYS_ON_DEFAULT
```

With this set, a trigger during any move discards the current move and stops the axis. `M120` and `M121` turn the behaviour on and off at runtime, so the behaviour can be proven before reflashing.

Three limits are worth knowing:

- It protects only the ends that have a switch fitted.
- After such a stop the position is no longer trustworthy, so re-home before relying on a coordinate.
- With default polling, response is bounded by the interrupt rate, so a fast axis overshoots the trigger point somewhat.

### A.4 Prove the switch before any motion

1. Build, flash and connect.
2. Send `M115` and confirm the build timestamp matches the firmware just flashed.
3. Send `M119` with the carriage clear of the switch. `x_min` should read `open`.
4. Press the switch by hand and send `M119` again. It should read `TRIGGERED`.

> **Do not home until both readings are correct.** A switch reading the wrong way round cannot stop the carriage.

If the two readings are swapped, flip `X_MIN_ENDSTOP_INVERTING`, rebuild and recheck.

### A.5 Home

> **Rest a finger near the switch before the first homing move.** The direction is unproven, and a geared axis is self-holding, so the carriage cannot be pulled clear by hand.

1. Send `G28 X`.
2. Watch the first centimetre of travel.
3. If the carriage moves toward the switch, let it finish. It touches, backs off, re-touches slowly, and stops a short distance clear.
4. If the carriage moves away from the switch, trigger the switch by hand to stop it. Set `INVERT_X_DIR true`, reflash, and repeat.
5. Send `M114`. X should read the backoff distance rather than zero.

---

## Variant B — Two limit switches

A second switch at the far end acts as a hard limit. Homing still uses one direction.

### B.1 Wire and configure

1. Wire the home-end switch as Variant A.
2. Wire the far-end switch to the X-MAX header, again between signal and ground.
3. Add the max plug and its measured polarity:

```cpp
#define USE_XMAX_PLUG
#define X_MAX_ENDSTOP_INVERTING true   // measure it, exactly as the min switch
```

### B.2 Enable the software limits

```cpp
#define MIN_SOFTWARE_ENDSTOPS
#define MAX_SOFTWARE_ENDSTOPS
```

Software limits refuse a move that would leave the configured travel. They are computed from the position the firmware believes it is at, so they are only as good as that belief. Once an axis skips steps, the firmware's coordinate frame no longer matches the machine, and the software limits are protecting a frame that has drifted. The physical switch is what remains.

Software limits also apply per axis, and only once that axis has been homed. An axis declared but never homed is not limited, which is the intended behaviour for a declared-only axis with no motor.

### B.3 Verify

1. Send `M119`.
2. Confirm `x_min` and `x_max` each read `open` when free and `TRIGGERED` when pressed.
3. Home with `G28 X`.

> A different two-switch arrangement puts two motors on one axis and squares a gantry against two switches. That belongs to a multi-motor axis rather than a single drive unit, and Guide 2 §2.5 covers it.

---

## 1.2 What the position report means after a stop

`M114` reports where the firmware believes the carriage is, which is not always where it is.

When an endstop triggers outside homing, Marlin records the trigger position and discards the current move, but nothing resets the logical position. The gcode layer has already advanced that position to the commanded target, so `M114` keeps reporting the target the machine never reached.

Marlin only re-synchronises position during homing. The rule that follows is short: **after any endstop hit outside `G28`, re-home before trusting a coordinate.**

---

## 1.3 A Z axis with no bed and no probe

On a Z axis, pin the homing height:

```cpp
#define Z_HOMING_HEIGHT 0
```

Left undefined, Marlin substitutes `Z_CLEARANCE_BETWEEN_PROBES`, which is 5 mm by default, and raises Z by that amount on every `G28`. On a machine with no bed and no probe the raise is unexplained and unwanted.

---

## 1.4 Changing method later

Moving between a switch and sensorless homing means rewiring as well as reconfiguring, because the two share one input pin. Guide 3 covers the change. Where both must be fitted at once and selectable without touching the wiring, the Advanced guide covers that arrangement.

---

**Next:** Guide 2 composes several validated units into a multi-axis machine, including a gantry with two motors on one axis.
