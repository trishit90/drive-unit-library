# Drive unit assembly guide

This guide covers the mechanical assembly of one drive unit, using the DU-220/540 reference instance as the worked example. Parts and quantities are in [BOM.md](BOM.md). Printed files and print settings are in [`../cad/README.md`](../cad/README.md). After the mechanism is built, continue with [`guides/Quick_Start_Guide.md`](guides/Quick_Start_Guide.md) to configure, flash and home the axis.

The parts referred to below are numbered in this view: (1) structural frame, (2) main-belt tensioner, (3) carriage, (4) reversible endstop mount, (5) adjustable motor mount, (6) transmission module, (7) cable-management adapter, (8) cross-brace.

![Exploded view of the drive unit parts, numbered 1 to 8](images/drive-unit-parts.png)

Source and limits. The content is derived from Sections 5.3 to 5.7 of the thesis (draft Chapters 1 to 7, v1). The thesis states the parts, fits and adjustments below. It does not publish torque values, belt-tension figures or a numeric running-fit criterion, so none are given here. The ordered 123-operation list used for the thesis operation count is held in the thesis appendix and is not reproduced in this file. The order of steps below follows the thesis description (frame first, modules after) and has not been timed or re-checked against a fresh build.

## Tools

Hex keys (button-head internal-hex screws are used throughout, apart from one M6 external-hex screw), a drill and a tap for the blind-joint features, and the appropriate drill for any printed hole that needs a controlled diameter. For stock preparation the thesis used a drop saw for the extrusion, scissors for the main belt and an angle grinder for the shaft. The exact drill and tap sizes are not stated in the thesis.

## 1. Prepare the stock

Cut the 2020 V-slot extrusion to 2 x 500 mm (longitudinal) and 2 x 130 mm (transverse). Cut the open GT2 main belt to 870 mm (approximately 2L - 130 mm for rails of length L). Cut the 5 mm steel shaft to 75 mm. Use the 210 mm closed-loop reduction belt as supplied. Print the PETG parts to the reference settings in [`../cad/README.md`](../cad/README.md), in the prescribed orientation, with the carriage printed face down. Pass printed holes that need a controlled final diameter through with the appropriate drill where the printed diameter requires it.

## 2. Assemble the frame

The frame is two 500 mm rails joined by two 130 mm transverse members, giving a 90 mm clear gap between the rails. The reference unit uses blind joints: drill access holes and tap the member ends, thread an M5 x 25 mm button-head screw into the end of one member so its head bears in the channel of the adjoining member, and tighten it through the clearance hole in the adjoining profile with a hex key. Commodity right-angle connectors are an alternative for the same frame geometry.

Assemble the frame before fitting any module. Modules that span both rails are introduced diagonally through the rail gap and rotated into place, so the end modules can be fitted and removed without dismantling the frame.

## 3. Build the carriage

Fit three 24 mm POM V-wheels to the printed carriage with M5 x 25 mm button-head screws, M5 nuts and spring washers. Two wheels are fixed and run on one rail. The third is mounted on the M5 eccentric adjuster and runs on the opposite rail. When fitting, turn the eccentric inward to minimise wheel spacing, place the carriage between the rails, then rotate the eccentric outward to apply preload until the three wheels give the required running fit. This is the single running-fit adjustment of the guidance system.

Secure each end of the main belt to the carriage with an M3 x 30 mm button-head through-bolt, washer and nut, so that the belt force acts along the printed layers. Leave final belt tension until the tensioner is fitted.

The moving machine interface is a pattern of 5 mm holes on 20 mm centres on the carriage.

## 4. Build the transmission module

Assemble the shaft stack in the open before closing the housing. From the driven-pulley end, the stack on the 5 mm x 75 mm shaft is: 625ZZ bearing, printed spacer, driven pulley, printed spacer, 625ZZ bearing, printed spacer, 20T output pulley, printed spacer, 625ZZ bearing. Each printed spacer is 2 mm thick with a 5 mm bore and 8 mm outside diameter. Slip the closed-loop reduction belt onto the driven pulley before the housing is closed.

The housing is split along the shaft plane. Lay the shaft stack into the bearing seats of the first half (light push or slip fit), then fit the second half, which captures the bearings. The two halves are polarised, and six M3 nuts seat in printed hex recesses. The seventh M3 x 30 mm screw and nut form the reaction adjustment used later to tension the reduction belt. Install the module on the frame with four M4 x 8 mm screws and T-nuts.

Select the driven pulley for the required ratio: 20T (1:1), 60T (3:1), 80T (4:1) or 100T (5:1). DU-220/540 uses 80T. The output pulley stays 20T, so one shaft revolution gives 40 mm of belt travel for every ratio.

## 5. Fit the motor and set the reduction belt

Fit the 20T motor pulley to the motor shaft, leaving its axial position adjustable. Attach the motor to the printed motor mount with two M3 x 6 mm screws, and place the mount on the extrusion with two M4 x 8 mm screws and T-nuts left loose. Set the motor pulley axially so the reduction belt runs in line with the driven pulley, then tighten its grub screws. Slide the motor mount along the extrusion to set the centre distance, using the reaction screw in the transmission module to push the mount away until the reduction belt reaches the required tension. Tighten the two frame screws to lock the mount. One motor face contacts the extrusion so heat can spread into the frame.

## 6. Fit the main-belt tensioner

The tensioner has a printed frame-mounted carrier, a moving idler and an adjustment knob. Build the idler with three 625ZZ bearings on an M5 x 25 mm screw and nut, with two 0.5 mm metal spacers in the stack. Capture the M6 x 20 mm external-hex screw in the knob and the M6 nut in the moving idler. Mount the carrier at the end of the frame opposite the transmission with four M4 x 8 mm screws and T-nuts. Route the main belt around the transmission output pulley and the idler, with both ends already secured at the carriage, then turn the knob to set the main-belt tension. The reduction and main belts are adjusted independently.

## 7. Fit the endstop

Attach the printed endstop mount (left- or right-hand variant, to suit the switch format) to the extrusion with one M4 x 8 mm screw and T-nut. Fix the mechanical switch to the mount with two M3 x 6 mm screws. Slide the mount along the extrusion to set the carriage home position, then tighten its screw. Switch polarity and homing direction are set in firmware (see the Quick Start and Guide 1).

## 8. Fit the cross-braces

Fit the printed cross-braces across the rails between the end modules, each with two M4 x 20 mm screws and T-nuts (four of each for two braces). DU-220/540 uses two. Brace number and position depend on axis length and cable routing. The braces also act as feet.

## 9. Optional cable management

Where required, fit the two identical printed adapters, one on the carriage as the moving attachment and one on the frame as the stationary attachment, with a commodity drag chain between them. This adds 17 assembly operations and is not fitted to DU-220/540.

## 10. Check before powering

Rotate the transmission by hand through the full travel. The carriage should move along the rails without binding, and the belts should seat on their pulleys without rubbing a flange. Re-check the eccentric preload, the reduction-belt tension and the main-belt tension. The thesis does not give numeric acceptance values for these, so judge them by feel and by the first-motion checks in the Quick Start.

Then wire the motor, endstop and 24 V supply and work through [`guides/Guide_0_Bench_Bringup.md`](guides/Guide_0_Bench_Bringup.md) and [`guides/Quick_Start_Guide.md`](guides/Quick_Start_Guide.md).

## Safety

The unit is driven by a 24 V stepper system and can pinch fingers or belts at the carriage, pulleys and tensioner. Keep hands clear while the controller is powered, and use the stop and release controls described in the Operator and guides when working near the mechanism.
