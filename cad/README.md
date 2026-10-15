# cad

Hardware design files for the drive unit. Everything in this folder is released under the CERN Open Hardware Licence Version 2, Permissive (`LICENSE` in this folder).

## Files

| File | Contents |
|---|---|
| `DriveUnit_V37.step` | STEP model of the drive unit, version 37 |
| `DriveUnitV37STLs/` | 3MF print files for the custom parts (see below). Despite the folder name, the files are 3MF; no STL files are included. |

## Print files

The individual part files carry the title stored in their 3MF metadata. Most read "DriveUnit v37". `BeltTightenerLeft`, `BeltTightenerRight` and `carriage2` read "DriveUnit2 v8", and `M6Knob` reads "m6Knob v3".

| File | Part |
|---|---|
| `Carriage.3mf`, `carriage2.3mf` | Carriage variants |
| `MotorBracket.3mf` | Motor bracket |
| `PulleyStackHolderTop.3mf`, `PulleyStackHolderBottom.3mf` | Pulley stack holder, top and bottom halves |
| `pulleyBrace.3mf` | Pulley brace |
| `beltTightner.3mf` | Belt tightener (original spelling retained in the file name) |
| `BeltTightenerLeft.3mf`, `BeltTightenerRight.3mf` | Handed belt tightener variants |
| `LimitSwitchLeft.3mf`, `LimitSwitchRight.3mf` | Limit switch mounts, left and right |
| `cableChainMount.3mf`, `stiffnerCableChain.3mf` | Cable chain mount and stiffener |
| `M6Knob.3mf` | M6 adjustment knob |

Two files are PrusaSlicer project plates that reference the part files above. `DriveUnitGantry.3mf` arranges 37 part instances (three each of the carriage, motor bracket, both pulley stack holders, belt tightener, pulley brace, stiffener and right limit switch; four cable chain mounts and four M6 knobs; two of each handed belt tightener; one `carriage2`). `refrence.3mf` arranges one of each of nine parts (stiffener, both pulley stack holders, carriage, belt tightener, motor bracket, M6 knob, pulley brace, right limit switch). `Print orientation.png` is a PrusaSlicer view of a plate showing the parts laid on their largest flat face.

## Print settings stored in the project plates

These values are read from the PrusaSlicer configuration inside `DriveUnitGantry.3mf` and `refrence.3mf`.

| Setting | Value |
|---|---|
| Printer profile | Prusa CORE One, 0.4 mm nozzle |
| Material profile | Generic PETG |
| Layer height | 0.25 mm |
| Perimeters | 6 |
| Top and bottom solid layers | 5 and 5 |
| Infill | 50 %, grid |
| Nozzle and bed temperature | 250 C and 85 C |
| Supports | off in `DriveUnitGantry.3mf`, on in `refrence.3mf` |

The part files do not carry bought-part information. See the bill of materials in `docs/` once it is added.
