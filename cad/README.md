# CAD

Hardware design files for the drive unit. Everything in this folder is released under the CERN Open Hardware Licence Version 2, Permissive (`LICENSE` in this folder).

## Files

| File | Contents |
|---|---|
| `DriveUnit_V37.step` | STEP model of the drive unit, version 37 |
| `DriveUnitV37STLs/` | 3MF print files for the custom parts |

## Print files

| File | Part |
|---|---|
| `Carriage.3mf`, `carriage2.3mf` | Carriage variants |
| `MotorBracket.3mf` | Motor bracket |
| `PulleyStackHolderTop.3mf`, `PulleyStackHolderBottom.3mf` | Pulley stack holder, top and bottom halves |
| `pulleyBrace.3mf` | Pulley brace |
| `beltTightner.3mf` | Belt tightener |
| `BeltTightenerLeft.3mf`, `BeltTightenerRight.3mf` | Handed belt tightener variants |
| `LimitSwitchLeft.3mf`, `LimitSwitchRight.3mf` | Limit switch mounts, left and right |
| `cableChainMount.3mf`, `stiffnerCableChain.3mf` | Cable chain mount and stiffener |
| `M6Knob.3mf` | M6 adjustment knob |

`DriveUnitGantry.3mf` is a PrusaSlicer project that references the part files above and arranges 37 part instances: three each of the carriage, motor bracket, both pulley stack holders, belt tightener, pulley brace, stiffener and right limit switch; four cable chain mounts and four M6 knobs; two of each handed belt tightener; and one `carriage2`. `Print orientation.png` is a PrusaSlicer view of a plate showing the parts laid on their largest flat face.

## Reference print settings

| Parameter | Reference setting |
|---|---|
| Printer | Prusa CORE One |
| Nozzle | 0.4 mm high-flow |
| Material | PETG |
| Layer height | 0.20 mm |
| Walls | 6 |
| Top and bottom layers | 5 and 5 |
| Infill | 50 % gyroid |
| Supports | Build-plate-only where required |
| Bridging | Permitted |
| Orientation | Prescribed orientations, as in the plate view |

One basic drive unit uses approximately 430 g of PETG and about 9 h 50 min of print time at these settings. Both depend on the printer, material profile and slicer.

Bought parts are listed in [docs/BOM.md](../docs/BOM.md).
