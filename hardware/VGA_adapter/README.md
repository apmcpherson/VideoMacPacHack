# VideoMacPac video adapter

These design materials are in the public domain, with no warranty whatsoever. See `documentation` on this repository for further design notes and `photos` for a photo of the assembled board.

## EAGLE files

* `VideoMacPac_adapter.sch`: EAGLE schematic
* `VideoMacPac_adapter.brd`: EAGLE PCB design
* `schematic.png`: image of the schematic

## Gerbers

This is a 2-layer PCB which can be fabricated from the files in the `gerbers` directory. It is a simple design which can be made on a mill with no soldermask as long as it has plated-through holes. The most important files are:

* `copper_top.gbr`: top copper
* `copper_bottom.gbr`: bottom copper
* `profile.gbr`: board outline
* `drill_1_16.xln`: Excellon drill file

## Bill of Materials

| Name    | Value                                 |
| ------- | ------------------------------------- |
| C1-C2   | 0.1uF 0603 capacitors                 |
| IC1     | 74LVC245 (20-SOIC) bus transceiver    |
| IC2     | MCP1700-3302T (SOT23) 3.3V regulator  |
| J1      | DB25 female                           |
| J2      | HD15 (VGA) female                     |
| R1-R3   | 133 ohm 0603 resistors                |
| R4-R6   | 150 ohm 0603 resistors                |
| R7      | 100 ohm 0603 resistor (could use 150) |