# Licences

This repository is not released under a single licence. Each top-level folder carries its own licence file, and the terms in that file apply to the contents of that folder only.

| Folder | Contents | Licence | SPDX identifier |
|---|---|---|---|
| `cad/` | Hardware design files: STEP model and 3MF print files | CERN Open Hardware Licence Version 2, Permissive | `CERN-OHL-P-2.0` |
| `firmware/` | Marlin configuration files | GNU General Public License, version 3 | `GPL-3.0-only` or `GPL-3.0-or-later`, per the headers in the files |
| `software/` | Drive Unit Calculator, Drive Unit Operator | MIT License | `MIT` |
| `docs/` | Assembly guide, BOM, wiring diagrams, sourcing notes | Creative Commons Attribution 4.0 International | `CC-BY-4.0` |
| `data/` | Vibration measurement records | Creative Commons Attribution 4.0 International | `CC-BY-4.0` |
| `data/` | Analysis script and G-code programmes | MIT License | `MIT` |

The licence texts are in `cad/LICENSE`, `firmware/LICENSE`, `software/LICENSE` and `docs/LICENSE`. The `data/` folder uses the texts in `docs/LICENSE` and `software/LICENSE`; `data/README.md` states which item takes which licence.

The Marlin configuration files are derived from GPLv3 material and cannot be relicensed. Files added to a folder take that folder's licence unless the file carries its own notice.

Third-party parts named in the bill of materials remain the property of their manufacturers and suppliers. Naming a part does not licence it.
