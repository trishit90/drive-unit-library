# Bill of materials: DU-220/540 reference instance

This is the mechanical bill of materials for the reference drive unit, DU-220/540: 220 mm usable travel, 540 mm overall length, 500 mm longitudinal rails, 130 mm transverse members, blind-jointed frame, 80T driven pulley (4:1), two cross-braces, no cable management. It is reproduced from Table 5.5 of the thesis (Section 5.8.1, draft Chapters 1 to 7, v1).

Scope. The table covers the drive-unit mechanism only. The controller, power supply, wiring and connectors are listed separately below. The motor is named because it was fitted to the reference unit. The drive-unit interface is defined by component family (2020 V-slot extrusion, 2 mm-pitch GT2, NEMA 17, 625ZZ bearings, compatible V-wheels), not by part number, so a compatible substitute from another supplier is permitted.

## Mechanical components

| Assembly | Component | Qty | Specification |
|---|---|---|---|
| Structural frame | Longitudinal extrusion | 2 | 2020 V-slot aluminium, 500 mm |
| Structural frame | Transverse extrusion | 2 | 2020 V-slot aluminium, 130 mm |
| Structural frame | Blind-joint screw | 4 | M5 × 25 mm button-head, internal hex |
| Moving carriage | Printed carriage | 1 | PETG |
| Moving carriage | V-wheel | 3 | 24 mm POM |
| Moving carriage | Eccentric adjuster | 1 | M5 V-wheel eccentric |
| Moving carriage | Wheel screw | 3 | M5 × 25 mm button-head, internal hex |
| Moving carriage | Wheel nut | 3 | M5 hex nut |
| Moving carriage | Spring washer | 3 | M5 |
| Moving carriage | Belt-retention screw | 2 | M3 × 30 mm button-head, internal hex |
| Moving carriage | Belt-retention washer | 2 | M3 |
| Moving carriage | Belt-retention nut | 2 | M3 hex nut |
| Motor mount | Printed motor mount | 1 | PETG |
| Motor mount | Stepper motor | 1 | LDO 42STH48-2804AC-R, NEMA 17, 1.8° |
| Motor mount | Motor pulley | 1 | 20T GT2, 5 mm bore, 10 mm belt width; supplied with grub screws |
| Motor mount | Motor-fastening screw | 2 | M3 × 6 mm button-head, internal hex |
| Motor mount | Frame-fastening screw | 2 | M4 × 8 mm button-head, internal hex |
| Motor mount | T-nut | 2 | M4 |
| Transmission module | Printed housing half A | 1 | PETG |
| Transmission module | Printed housing half B | 1 | PETG |
| Transmission module | Printed axial spacer | 4 | 2 mm thick, 5 mm bore, 8 mm outside diameter |
| Transmission module | Transmission shaft | 1 | Steel, Ø5 × 75 mm |
| Transmission module | Bearing | 3 | 625ZZ |
| Transmission module | Driven pulley | 1 | 80T GT2, 5 mm bore, 10 mm belt width; supplied with grub screws |
| Transmission module | Output pulley | 1 | 20T GT2, 5 mm bore, 10 mm belt width; supplied with grub screws |
| Transmission module | Reduction belt | 1 | 210 mm closed-loop GT2, 10 mm wide |
| Transmission module | Housing/reaction screw | 7 | M3 × 30 mm button-head, internal hex |
| Transmission module | Housing/reaction nut | 7 | M3 hex nut |
| Transmission module | Frame-fastening screw | 4 | M4 × 8 mm button-head, internal hex |
| Transmission module | T-nut | 4 | M4 |
| Main-belt tensioner | Printed frame-mounted carrier | 1 | PETG |
| Main-belt tensioner | Printed moving idler | 1 | PETG |
| Main-belt tensioner | Printed adjustment knob | 1 | PETG |
| Main-belt tensioner | Bearing | 3 | 625ZZ |
| Main-belt tensioner | Bearing spacer | 2 | Metal, 0.5 mm thick, 5 mm bore, 8 mm outside diameter |
| Main-belt tensioner | Idler screw | 1 | M5 × 25 mm button-head, internal hex |
| Main-belt tensioner | Idler nut | 1 | M5 hex nut |
| Main-belt tensioner | Adjustment screw | 1 | M6 × 20 mm external hex-head |
| Main-belt tensioner | Captive adjustment nut | 1 | M6 hex nut |
| Main-belt tensioner | Frame-fastening screw | 4 | M4 × 8 mm button-head, internal hex |
| Main-belt tensioner | T-nut | 4 | M4 |
| Endstop assembly | Printed endstop mount | 1 | PETG, appropriate left- or right-hand variant |
| Endstop assembly | Mechanical endstop switch | 1 | Supported mechanical module format |
| Endstop assembly | Switch-fastening screw | 2 | M3 × 6 mm button-head, internal hex |
| Endstop assembly | Frame-fastening screw | 1 | M4 × 8 mm button-head, internal hex |
| Endstop assembly | T-nut | 1 | M4 |
| Cross-braces | Printed cross-brace | 2 | PETG |
| Cross-braces | Frame-fastening screw | 4 | M4 × 20 mm button-head, internal hex |
| Cross-braces | T-nut | 4 | M4 |
| Main belt | Main carriage belt | 1 | 870 mm open GT2, 10 mm wide |

## Totals

The reference instance contains 14 printed parts and 24 bought non-fastener parts, giving 38 primary mechanical parts. The assembly uses 71 separate fastening elements, for a total of 109 mechanical components. The grub screws supplied with the three pulleys are treated as part of the purchased pulleys. By type, the thesis counts 10 printed part types, 13 bought non-fastener types and 12 fastener types, so the primary mechanical system is 38 parts across 23 primary part types. The thesis counts 123 mechanical assembly operations for this instance under its Chapter 4 definition.

The quantities in the table above were summed from the table rows and reproduce the thesis totals of 14, 24, 71 and 109.

## Optional cable management (not fitted to DU-220/540)

| Component | Qty | Specification |
|---|---|---|
| Printed cable-management adapter | 2 | Identical PETG parts |
| Drag chain | 1 | Commodity cable chain |
| Adapter screw | 4 | M3 x 6 mm button-head, internal hex |
| Mounting screw | 4 | M4 x 20 mm button-head, internal hex |
| Mounting nut | 4 | M4 hex nut |

Fitting this assembly adds 17 mechanical assembly operations (thesis Table 5.6).

## Control hardware (reference implementation)

The thesis names these items in Section 5.6.1 and Table 5.7 but does not give a full electrical bill of materials. Items marked "not stated" have no value in the thesis text and need to be added by the builder or author.

| Item | Reference value |
|---|---|
| Controller | BigTreeTech Octopus V1.1 |
| Stepper driver | TMC2209 (UART); the Calculator also supports TMC5160 (SPI) |
| Run current | 1.4 A RMS |
| Microstepping | 1/16 |
| Supply voltage | 24 V |
| Power supply rating | Not stated |
| Motor connector | JST-PH (fitted to the project motors as supplied) |
| Wiring, endstop cable, USB cable, microSD card | Not stated |
| Firmware | Marlin 2.1.2 (the files in this repository are for 2.1.2.8) |

## Stock preparation (thesis Table 5.2)

| Item | Supplied form | Preparation for DU-220/540 |
|---|---|---|
| Printed components | PETG filament | FDM on Prusa CORE One using the settings in `cad/README.md` |
| 2020 V-slot extrusion | 2.4 m stock length | Cut to 2 x 500 mm and 2 x 130 mm using a drop saw |
| Blind-joint features | Cut extrusion members | Drill access holes and tap required extrusion ends |
| Main belt | Open 10 mm GT2 belt supplied in 10 m stock | Cut to 870 mm using scissors |
| Reduction belt | 210 mm closed-loop GT2 belt | Used as supplied |
| Transmission shaft | 5 mm steel rod | Cut to 75 mm using an angle grinder |
| Printed holes requiring controlled final diameter | As printed | Passed through with the appropriate drill where required |
| Bearings, pulleys, wheels and fasteners | Commodity components | Used as supplied |

## Changing the instance

Four mechanical choices define an instance: longitudinal extrusion cut length, main-belt cut length, frame-joint type (blind joint or commodity right-angle connector) and cross-brace quantity. The driven pulley sets the ratio: 20T, 60T, 80T or 100T give 1:1, 3:1, 4:1 and 5:1, and the Calculator accepts other tooth counts from 20T to 100T. The 210 mm reduction belt is retained across the standard pulley options.

Open main-belt length is approximately B = 2L - 130 mm, where L is the longitudinal extrusion length (thesis equation 5.4). For L = 500 mm this gives 870 mm. The design range is approximately 300 mm to 2000 mm of longitudinal extrusion. The rule is a preparation estimate; final tension is set on the assembled axis with the tensioner.

## Printed parts and files in `cad/`

The thesis lists printed parts by function; the files in `cad/DriveUnitV37STLs/` are named differently. The mapping below is inferred from file names and has not been confirmed.

| Printed part in the table | Probable file (unconfirmed) |
|---|---|
| Printed carriage | `Carriage.3mf` |
| Printed motor mount | `MotorBracket.3mf` |
| Printed housing half A and B | `PulleyStackHolderTop.3mf`, `PulleyStackHolderBottom.3mf` |
| Printed adjustment knob | `M6Knob.3mf` |
| Printed endstop mount (left or right) | `LimitSwitchLeft.3mf`, `LimitSwitchRight.3mf` |
| Printed cross-brace | `pulleyBrace.3mf` |
| Tensioner carrier and moving idler | `beltTightner.3mf` (unconfirmed) |
| Printed axial spacer (4) | No matching file identified |
