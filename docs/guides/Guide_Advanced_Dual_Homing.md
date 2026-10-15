# Advanced — a limit switch and sensorless homing on one axis

**Scope.** Carry both a physical limit switch and sensorless StallGuard homing on the same axis at the same time, wired so that changing the active method needs no rewiring. Guide 1 and Guide 3 keep the two apart because they share one input pin; this guide separates them onto two pins.

**Firmware target.** Marlin 2.1.2.8, Octopus V1.1 pins.

---

## A.1 What this arrangement does and does not do

**Which method homes the axis is a compile-time choice**, set by whether `SENSORLESS_HOMING` is enabled. No G-code switches an axis between homing on the switch and homing on stall. No runtime toggle exists.

What it does give is worth having anyway:

- Changing the active method is a one-line edit and a reflash, with no wires moved and no jumper touched.
- The switch stays live as a hardware limit while sensorless does the day-to-day homing, so a stall that fails to trigger still meets a real switch.

That second point is the stronger reason. Sensorless homing detects a collision it has already had; the switch is the backstop for the case where it does not detect one.

---

## A.2 Why a second input is needed

Enabling sensorless homing routes the driver's stall output onto the axis's standard endstop pin, which for X is PG6 on the X-STOP header. A switch on the same header would contend with that signal.

The answer is to put the switch on a different input. Marlin's board file does this automatically: when a stall sensitivity is defined for the axis, it assigns the stall line to the standard endstop pin and remaps the physical switch input to a spare detector header.

> **Confirm the remapped pin against the board file being compiled.** Open `pins_BTT_OCTOPUS_V1_common.h`, find the block guarded by `X_STALL_SENSITIVITY`, and read which header the physical input moves to. Wire the switch to that header, not to X-STOP.

---

## A.3 Wire both

1. Disconnect all power.
2. Fit the DIAG jumper for the axis's slot, connecting the stall output to X-STOP.
3. Wire the limit switch, normally closed between signal and ground, to the spare detector header confirmed in §A.2.
4. Confirm the switch is not wired to X-STOP.

Both signals now reach the board on separate pins.

---

## A.4 Pin the inputs so they never move

The automatic remap is convenient but it moves the switch input depending on whether sensorless is enabled. Toggling `SENSORLESS_HOMING` would then move the pin under a switch that has not moved, which is exactly the rewiring this guide exists to avoid.

State both pins explicitly, so neither depends on the sensorless setting:

```cpp
#define X_DIAG_PIN PG6      // stall output, always on X-STOP
#define X_MIN_PIN  PG12     // physical switch, always on the spare header
```

> **Treat both pin numbers as placeholders until they are confirmed against the board file in use.** They differ between board revisions.

With the pins fixed, changing the homing method is a single edit:

```cpp
#define X_STOP_PIN X_MIN_PIN    // home on the switch
// #define X_STOP_PIN X_DIAG_PIN   // home on stall
```

Enable or disable `SENSORLESS_HOMING` to match, then rebuild and reflash. The wires never move.

---

## A.5 Configure

In `Configuration_adv.h`, for sensorless as the active method:

```cpp
#define SENSORLESS_HOMING

#if EITHER(SENSORLESS_HOMING, SENSORLESS_PROBING)
  #define X_STALL_SENSITIVITY 100   // the tuned value; the scale differs by driver (Guide 3 §3.3)
  #define IMPROVE_HOMING_RELIABILITY
#endif

#define X_HOME_BUMP_MM 0
#define X_CURRENT_HOME 700
#define ENDSTOPS_ALWAYS_ON_DEFAULT
```

In `Configuration.h`, keep the switch enabled so Marlin reads it as a real endstop:

```cpp
#define USE_XMIN_PLUG
#define ENDSTOPPULLUPS
#define X_MIN_ENDSTOP_INVERTING true   // measured at the pin, as Guide 1 §A.2
#define MIN_SOFTWARE_ENDSTOPS
#define MAX_SOFTWARE_ENDSTOPS
```

`ENDSTOPS_ALWAYS_ON_DEFAULT` matters more here than anywhere else in the set. Without it the switch is read only during homing, which removes the backstop that is the whole point of the arrangement.

---

## A.6 Verify both inputs independently

1. Build, flash and connect.
2. Send `M115` and confirm the build timestamp.
3. Send `M119` with the carriage clear and the switch released.
4. Press the switch by hand and send `M119` again.

   ✓ **Result:** the switch state changes on its own, independently of the stall line.

5. Send `G28 X` and confirm the axis homes by the selected method.
6. With sensorless active, confirm the switch still reads correctly afterwards.

Where the switch state does not change under a finger, the wire is on the wrong header. Return to §A.2.

---

## A.7 Tuning order

Order matters, because each step invalidates the one before it.

1. Set the run current first (Guide 0 §0.8). StallGuard reads motor load, which scales with current.
2. Set the homing feedrate.
3. Tune the stall sensitivity with `M914`, at that current and that feedrate (Guide 3 §3.4).
4. Re-tune whenever either changes.

Keep `X_CURRENT_HOME` reduced so the stall is gentle and repeatable.

Remember the scale direction differs by driver: on a TMC2209 the range is 0 to 255 and higher is more sensitive; on a TMC5160 it is −64 to +63 and lower is more sensitive.

---

## A.8 When this is worth the complexity

The dual arrangement earns its cost in two situations. The first is where sensorless homing suits daily use, having no switch to foul or wear, but a hardware limit is still wanted as a backstop. The second is while the choice between the two methods is still open, and both must be compared on the same axis without rebuilding the wiring each time.

For a settled machine, one physical switch (Guide 1) or plain sensorless homing (Guide 3) is simpler and usually enough.

---

*End of the set: Guide 0 (bench bring-up) → Guide 1 (limit switches) → Guide 2 (multi-axis machine) → Guide 3 (sensorless homing) → Advanced (both on one axis). The Quick start runs alongside Guide 0 as the fast path.*
