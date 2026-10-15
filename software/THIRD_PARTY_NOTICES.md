# Third-party notices

`drive_unit_calculator.html` and `drive_unit_panel.html` embed three typefaces (the Calculator carries three font-face blocks, as does the Operator panel) as base64 data in their stylesheets, so each page makes no external requests. The two HTML files are released under the MIT License (see `LICENSE`). The embedded fonts are not covered by that licence. They remain under the SIL Open Font License, Version 1.1, whose full text is in `OFL.txt` in this folder.

| Typeface | Role in the pages | Copyright line read from the embedded font |
|---|---|---|
| Jura | Display | Copyright 2019 The Jura Project Authors (https://github.com/ossobuffo/jura) |
| Share Tech Mono | Body and data | Copyright (c) 2012, Carrois Type Design, Ralph du Carrois (post@carrois.com www.carrois.com), with Reserved Font Name 'Share' |
| B612 Mono | Per-glyph fallback for characters Share Tech Mono lacks | Copyright 2012 The B612 Project Authors (https://github.com/polarsys/b612) |

The copyright lines above were read from the name tables of the fonts as embedded. Only the Share Tech Mono table states the licence (SIL Open Font License 1.1). The Jura and B612 tables carry no licence field, so their OFL 1.1 status is taken from the upstream projects and has not been re-checked against the embedded files.
