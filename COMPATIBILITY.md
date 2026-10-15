# Compatibility

The files in this repository were produced and checked as a set. Mixing them with other versions is outside what was checked.

## Versions that belong together

| Component | Version in this repository | Where it is stated |
|---|---|---|
| Hardware design | DriveUnit V37 (STEP model) | `cad/DriveUnit_V37.step`; most 3MF part files carry the title "DriveUnit v37" |
| Drive Unit Calculator | v4 | `software/drive_unit_calculator.html`; its output is tagged `drive_unit_calculator_v4` |
| Drive Unit Operator | v10 | `software/drive_unit_panel.html`; the panel's ready message reads "panel ready (v10)" |
| Firmware | Marlin 2.1.2.8 | Calculator, Operator header, and every guide in `docs/guides` |
| Controller | BigTreeTech Octopus V1.1 | Calculator default, Operator header, guides |
| Stepper driver | TMC2209 (UART) for the reference unit; TMC5160 (SPI) also supported by the Calculator | Guides, Calculator |
| Print settings | Thesis Table 5.1 | `cad/README.md` |
| Example configuration | `firmware/example_config.ini`, a Marlin `config.ini` for the reference setup | `firmware/` |

Mixed part-file versions are intended. `BeltTightenerLeft`, `BeltTightenerRight` and `carriage2` carry the title "DriveUnit2 v8", and `M6Knob` carries "m6Knob v3"; the other 3MF files are titled "DriveUnit v37".

## Notes

The thesis describes the reference build as Marlin 2.1.2. The guides, Calculator and Operator in this repository state 2.1.2.8, which is the tree the files were produced against.

The Operator runs in a desktop browser that supports Web Serial. As far as is known, that means current Chromium-based browsers (for example Chrome and Edge); this was not re-tested across browsers for this release. Page-level checks of both HTML tools used a mocked serial port, not a physical board.

The Operator guide in the project history was written for an earlier panel version and is not included. The guides in `docs/guides` were written for Marlin 2.1.2.8 on the Octopus V1.1 and should not be mixed with instructions written for another Marlin version.

Marlin configuration files are GPLv3 and are kept in `firmware/`. See [LICENSE.md](LICENSE.md) for the licence of each folder.
