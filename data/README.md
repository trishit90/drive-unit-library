# data

`vibration_data.zip` holds the end-effector vibration records and analysis code for the Chapter 7 case study (a reconstructed three-axis gantry compared with the published baseline gantry). The archive is released unmodified. Its own README, inside the zip at `ch7_vibration/README.md`, lists the records and gives the commands that reproduce the published values.

## Contents of the archive

- `ch7_vibration/data/`: four TDMS records (one baseline-gantry record and three reconstructed-gantry records)
- `ch7_vibration/programmes/`: the two G-code motion programmes as run
- `ch7_vibration/vibration_analysis.py`: the analysis script
- `ch7_vibration/README.md`: record register and reproduction instructions

The baseline gantry was recorded once and has since been dismantled, so its record cannot be repeated. Four further records excluded from the analysis are named in the thesis and are available on request.

## Licences

This folder carries two licences. Neither is the licence of the other folders in this repository; see `LICENSE.md` at the repository root.

| Item | Licence |
|---|---|
| TDMS measurement records | Creative Commons Attribution 4.0 International. Full text: `../docs/LICENSE` |
| `vibration_analysis.py` and the G-code programmes | MIT License. Full text: `../software/LICENSE` |

## Integrity check

SHA-256 of `vibration_data.zip`:

```
12d923839ebd7463bf66e6dac45ee0ae71c1edc3d34787acc94f077c67ee14db
```

In PowerShell: `Get-FileHash vibration_data.zip -Algorithm SHA256`
