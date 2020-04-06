# VideoMacPacHack

The following are implementation notes on cloning the Video Mac Pac PDS expansion card. The schematic and board layout can be found in the `hardware/VideoMacPac/` directory in this respository.

## Schematic

The schematic of the VideoMacPacHack conforms as closely as possible to the original VMP, with a few exceptions:

### Address decoder

On the VMP, a `GAL16V8` implements a relatively straightforward address decoder with 10 inputs (9 address lines plus an active sense signal) and 1 output (to the CPLD `WRT` pin). To avoid the need to recreate this custom IC, standard logic can be used instead. The basis of the current design is a `74F521` 8-bit address decoder. In addition to its 8 logic inputs, it has a 9th `OE/` input which can be used for one additional bit of decoding. 

However, one further bit is still required to replicate the functionality of the GAL. Because board space does not allow another DIP IC, a single-gate SMT IC is used instead: `74AHCT1G00`, a single 2-input NAND gate.

### Power switch

Miniature relays have been supplanted by analog power switch ICs. This design uses a `TPS2030` IC to achieve the same effect as the original relay. A DIP package is used in the VMPH design. However, SMT versions of the part appear to be more widely available and can be used with a DIP-SMT adapter.

### 74F244

This is a place where the original design should not be changed.

Initial experiments exchanged the `74F244` buffer IC for a nearly-equivalent `74HC244`. However, we found that the longer and more variable propagation delay on the `74HC244` caused vertical edges to show colour tints, as the edges on the three video channels were no longer aligned (the R, G and B lines have separate logic signals from the CPLD). The original 74F244 does not exhibit this problem.

## PCB Layout

The mechanical layout of the PCB follows the original VideoMacPac design, which is in turn based on Apple’s specifications in the Guide to the Macintosh Family Hardware. The component layout is broadly similar to the original.

The PCB is a 4-layer design, like the original VMP. The trace routing is electrically identical but does not follow the same geometry as the original. The original VideoMacPac design largely avoids vias, with traces crossing layers at existing through-hole pads. The VMPH alternative routing was done relatively quickly and has not sought to optimise the layout in this way.

The VMPH design attempts to use through-hole parts wherever possible. It was necessary to make a single exception for space reasons: U3 (`74AHCT1G00`), a single NAND gate which is used to provide one extra input to the address decoder in U2.

On the other hand, the video adapter dongle is made with SMT ICs and passives for compactness.

## Notes on Bill of Materials 

The VideoMacPacHack requires the following historical parts, which can sometimes be found as new old stock:

* 1x `XC2018-100-PC84` (84-pin PLCC): Xilinx CPLD
* 4x `MT42C4064Z-12` (24-pin ZIP): VRAM
* 5V 4-pin oscillator cans: 36MHz, 50.35MHz, 57.2832MHz (see below)

### CPLD

A `XC2018-70` was used in place of the `XC2018-100` with no adverse effects.

### VRAM and resistor networks

A `MT42C4064Z-10` was used in place of the `MT42C4064Z-12` in prototypes with no adverse effects.

The VRAM connects to the PDS data lines, but most signals go to the CPLD by way of 47 ohm resistors. On the VMP, 4 SIP resistor networks are used: two with 4 elements, two with 3 elements. If 3-element resistor networks are hard to find, 4-element versions can be used in those slots with the two pins on the end cut off.

### Oscillators

5V DIP oscillator cans are no longer available in as many frequencies as they once were. New old stock may need to be found for these frequencies. In most cases, similar but non-identical frequencies can be used with the effect of changing the refresh rate of the resolution. Driving a modern LCD monitor this would likely not be a problem. 

There appear to be limits to how far the frequencies can be stretched: for example, putting a 50MHz crystal in place of the 36MHz crystal produces no output at the affected resolutions.

