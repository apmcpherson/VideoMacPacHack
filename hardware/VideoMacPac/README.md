# VideoMacPacHack

These design materials are in the public domain, with no warranty whatsoever. See `documentation` on this repository for further design notes.

## Kicad files

* `VideoMacPac.pro`: Kicad project
* `VideoMacPac.sch`: Kicad schematic
* `VideoMacPac.kicad_pcb`: Kicad PCB design
* `VideoMacPac_schematic.pdf`: Schematic in PDF format

## Gerbers

This a 4-layer PCB which can be fabricated from the files in the `gerbers` directory.

* `VideoMacPac-F.Cu.gbr`: top copper
* `VideoMacPac-In1.Cu.gbr`: inner layer 1 copper
* `VideoMacPac-In2.Cu.gbr`: inner layer 2 copper
* `VideoMacPac-B.Cu.gbr`: bottom copper
* `VideoMacPac-F.SilkS.gbr`: top silkscreen
* `VideoMacPac-F.Mask.gbr`: top soldermask
* `VideoMacPac-B.Mask.gbr`: bottom soldermask
* `VideoMacPac-Edge.Cuts.gbr`: board outline
* `VideoMacPac.drl`: Excellon drill file

## Bill of Materials

| Name    | Value                                |
| ------- | ------------------------------------ |
| C1-C8   | 0.1uF axial capacitors               |
| J1      | 20-pin IDC header                    |
| J2      | 96-pin right-angle EuroDIN           |
| R1-R4   | 4.7k resistors                       |
| RN1-RN2 | 3x47 ohm SIP resistor networks       |
| RN3-RN4 | 4x47 ohm SIP resistor networks       |
| U1      | XC2018-100-PC84 (faster can be used) |
| U2      | 74F521 (DIP)                         |
| U3      | 74AHCT1G00 (SOT23)                   |
| U4      | 74F244 (DIP) -- do not substitute    |
| U5-U8   | MT42C4064Z-12 (faster can be used)   |
| U9      | TPS2030 (DIP)                        |
| X1      | 36MHz 4-pin oscillator               |
| X2      | 50.35MHz 4-pin oscillator            |
| X3      | 57.2832MHz 4-pin oscillator          |