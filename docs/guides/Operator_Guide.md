# Operator Guide

The Drive Unit Operator (`software/drive_unit_panel.html`, panel v10) is a single HTML file that talks to a Marlin 2.1.2.8 board over USB. It is used for commissioning and for routine motion: jogging, homing, going to saved positions, switching outputs and running repeatable sequences. It uses the Web Serial API, so it runs in desktop Chrome, Edge and other Chromium browsers. Firefox and Safari do not provide the API.

Open the file in the browser, plug the board in over USB, and press Connect. On Linux, add the user to the `dialout` group first, then log out and in again.

## Connecting

The panel opens the serial port at 115200 baud and reads the board identity (M115), settings (M503) and soft endstops (M211). It then sends M18 S0 to disable the idle stepper timeout, and energises all steppers. Steppers stay energised after connecting, including any axis released in an earlier session, so keep hands clear of the mechanism when pressing Connect. While connected and idle, the panel polls the position (M114) every 3 seconds.

Axis names, jog feeds, travel clamps and output channels are stored per machine, keyed to the identity the board reports, and are restored when that machine is reconnected. Storage is in the browser (localStorage). Use Export everything (JSON) on the Setup tab to keep a backup.

## Stops

- **STOP** sends M410. Motion halts at once, outputs are switched off and the machine must be re-homed.
- **Soft Stop** applies to a running sequence. The first press stops at the end of the current cycle, the second after the current step.
- **hold for KILL** sends M112 after the button has been held for 1.2 seconds. The firmware halts and stops responding. Power-cycle the board or press its reset button, reconnect, and re-home before any motion.

## Run tab

Motion shows the position readout, the step-size selector (default 0.1, 1, 10 and 50 mm), and the jog controls. Keyboard: Left and Right move the first axis, Up and Down the second, Page Up and Page Down the third, each by the selected step size. Keys are ignored while typing in a field. Jog feeds come from the profile and are limited to the firmware M203 maxima once these have been read. Home all homes the axes the panel knows about; it always names them, so an axis declared in firmware only for Marlin's axis count is not homed.

Waypoints are saved positions. Teach here stores the current position, and typed coordinates can be added. Waypoints hold positions only; feeds and outputs belong to sequence steps.

Aux holds output channels that drive Marlin fan outputs (M106 P&lt;index&gt; S255 to switch on, M107 P&lt;index&gt; to switch off). All channels are switched off on every stop. Right-click a channel to remove it.

Sequences are ordered lists of steps: move to a waypoint, dwell, output on or off, home, wait for the operator, or a single G-code line. A sequence runs once, a set number of times, or until stopped. Clone template offers four starting points: a repeatability sequence (home, move to a point, dwell), a speed ladder between two waypoints with rising feed, an acceleration ladder (M204), and a motor-current ladder (M906).

## Setup tab

Commission is ordered to match the Quick Start. Declare the current position as the origin (G92) to enable jog and go-to before endstops exist; a banner stays until a real G28. Check soft endstops, release motors (M18), read endstops (M119) and read the firmware values the panel is working from.

Verify checks steps/mm by command and measurement: move an axis a set distance, measure what it travelled, and compare. Steps/mm for the gearsets is 4 times the gearset tooth count: 100T is 400, 80T is 320, 60T is 240 and direct drive is 80. A small deviation points to mechanics such as belt tension or grub screws. A large one usually means the wrong gearset is declared in firmware; when the implied value matches another known gearset, the panel says so and gives the M92 line. Writing M92 is done from the console.

Travel capture records the usable far end of an axis by jogging to it and marking it. This sets the panel clamp and gives the define for the next firmware build. Calculator profile imports the Calculator's profile JSON so predicted steps/mm is cross-checked against the board at every connect.

Profile sets axis names, enabled axes, jog feeds and jog increments (comma-separated). Data exports and imports everything as JSON.

## Console

The serial console shows everything sent and received. Filters hide ok/busy lines and the automatic M114 poll. The log can be cleared or downloaded, and G-code can be typed and sent directly.
