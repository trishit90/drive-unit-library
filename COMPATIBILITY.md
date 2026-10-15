# Compatibility

The files in this repository are intended to be used as a set.

## Versions that belong together

| Component | Version in this repository | Where it is stated |
|---|---|---|
| Hardware design | DriveUnit V37 (STEP model) | `cad/DriveUnit_V37.step`; most 3MF part files carry the title "DriveUnit v37" |
| Drive Unit Calculator | v4 | `software/drive_unit_calculator.html`; its output is tagged `drive_unit_calculator_v4` |
| Drive Unit Operator | v10 | `software/drive_unit_panel.html`; the panel's ready message reads "panel ready (v10)" |
| Firmware | Marlin 2.1.2.8 | Calculator, Operator header, and every guide in `docs/guides` |
| Controller | BigTreeTech Octopus V1.1 | Calculator default, Operator header, guides |
| Stepper driver | TMC2209 (UART) for the reference unit; TMC5160 (SPI) also supported by the Calculator | Guides, Calculator |
| Print settings | Reference print settings | `cad/README.md` |
| Example configuration | `firmware/example_config.ini`, a Marlin `config.ini` for the reference setup | `firmware/` |

Mixed part-file versions are intended. `BeltTightenerLeft`, `BeltTightenerRight` and `carriage2` carry the title "DriveUnit2 v8", and `M6Knob` carries "m6Knob v3"; the other 3MF files are titled "DriveUnit v37".

## Notes

The guides, Calculator and Operator target Marlin 2.1.2.8.

The Operator runs in a desktop browser that supports Web Serial. That means current Chromium-based browsers such as Chrome and Edge.

The guides in `docs/guides` are written for Marlin 2.1.2.8 on the Octopus V1.1 and should not be mixed with instructions for another Marlin version.

Marlin configuration files are GPLv3 and are kept in `firmware/`. See [LICENSE.md](LICENSE.md) for the licence of each folder.
