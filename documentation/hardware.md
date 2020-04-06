# Video Mac Pac hardware

The following document is based on an analysis and reverse engineering of the Video Mac Pac board. To date we have not found any original documentation on this design.

## Operation overview

The Video Mac Pac, designed in 1991 by Computer Care Inc., is a PDS expansion card for the Macintosh Portable (M5120 and M5126) that provides a second video output in addition to the Portable’s built-in screen. It supports 1-bit video at several resolutions:

* 640x872 (Apple Portait Display)
* 640x872 (Sunkyung FPD)
* 640x872 (Samsung FPD)
* 640x480 (Apple Monochrome Display)
* 640x480 (VGA)
* 800x600 (Super VGA)
* 528x348 (Mono TTL)
* 272x208 (TV/VCR)

These resolutions are selectable through the Video Mac Pac control panel which must be installed for the card to work.

The hardware design and software drivers appear to be closely related to the Lapis DisplayServer-SE video card. The card is based on a Xilinx CPLD whose firmware is loaded at boot by the control panel, along with VRAM and some glue logic for interfacing to the PDS slot and to the display.

## CPLD

The Video Mac Pac is based on the Xilinx `XC2018-100PC84` CPLD, which has the following specifications: 

* 84-pin PLCC package 
* 5V logic
* 1800 gates
* 74 I/Os
* 100ns toggle rate 

The size of a program configuration on the XC2018 is 17878 bits. There is no non-volatile storage either on the XC2018 or elsewhere on the Video Mac Pac, meaning that the CPLD configuration is downloaded by the Macintosh on power-up.

On the VMP, the XC2018 mode pins `M0, M1, M2` are configured as `101`, corresponding to Peripheral mode. In this mode, the program is clocked in 1 bit at a time on the `DIN` and `WRT` pins.

For comparison, the Lapis DisplayServer-SE is also based on the XC2018. The SE-DPD (capable of driving dual page displays) instead uses the `XC3020-70-PC84`. 

## VRAM

The VMP uses four Micron `MT42C4064Z-10` 64kx4 VRAM ICs. These are in a 24-pin ZIP package (i.e. SIP with staggered leads). Each VRAM chip handles 4 bits of a 16-bit data word. 

The CPLD maps the VRAM to the memory address space from `$720000` to `$73FFFF` (128k bytes in total).

## GAL

The CPLD must be programmed by the Macintosh using the Video Mac Pac control panel. A `GAL16V8-25LNC` IC on the board implements an address decoder for address `$740200` (binary `0111 010x xxxx xx1x xxxx xxxx`). 

The GAL also takes as input a sense line (active high) from the external video dongle. When the dongle is connected, address `$740200` is matched and `AS/` is low, the output (tied to the `WRT` pin on the CPLD) goes low. This feature is used to program the CPLD after power-on, after which point further communication takes place directly to the CPLD via the bus on the PDS slot.

If the dongle is not connected when the control panel loads, the CPLD will not be programmed and the video card will not operate until a restart takes place.

## PDS slot

See Apple's *Designing Cards and Drivers for the Macintosh Family* for the pinout of the Macintosh Portable PDS slot. The following pins of the PDS slot connect to the CPLD:

* Clock signal `16M`
* All address lines `A23-A0`
* Data line `D0` (connected to `DIN` to program the CPLD)
* Address strobe `/AS`
* Data strobes `/UDS` and `/LDS`
* Read/write signal `R/W`
* Acknowledge signals `/DTACK` and `/EXT.DTACK`
* Reset signal `/SYS.RST`

The following parts of the PDS slot connect directly to the VRAM:

* Data lines `D15-D0`, 4 bits per IC

All other VRAM pins are connected via the CPLD.

## Relay

The VMP contains a miniature 5V relay which is switched by way of the PDS signal `5/0V`. This powers down the card when the Portable goes to sleep.

When the Portable wakes from sleep, the CPLD program will be lost and it appears that a power cycle is required to enable the external screen.

## Oscillators

The VMP has 3 oscillator cans: 36MHz, 50.35MHz, 57.2832MHz. The 36MHz oscillator is socketed, while the other two are soldered. One of the three oscillator signals is chosen by the CPLD depending on the screen resolution. These provide the pixel clock for the video output.

The oscillators are not powered directly on the board. Rather, power to the oscillators is routed to a pin on the external video dongle connector, and the dongle routes the power back to the oscillators. This appears to be a power saving measure when the dongle is not connected.

In addition to the three oscillators, the card receives a 16MHz clock from the PDS slot.

## Outputs

The CPLD generates eight output logic signals which are buffered through a `74F244` octal buffer. These signals include:

* Three video signals. Our testing showed that these three always generate the same signal, as the card is 1-bit monochrome, but perhaps keeping separate signals was intended for a future design which would be full colour.
* Horizontal and vertical sync
* Three other signals which measure as constant high or low depending on the chosen resolution. These may have performed control functions for an external dongle (now lost).

The signals appear on a 20-pin header with the following pinout:

| pin | description    | pin | description |
| --- | -------------- | --- | ----------- |
| 1   | oscillator +5V | 2   | +5V out     |
| 3   | ground         | 4   | sense input |
| 5   | ground         | 6   | HSYNC       |
| 7   | ground         | 8   | VSYNC       |
| 9   | ground         | 10  | unknown 1   |
| 11  | ground         | 12  | video 1     |
| 13  | ground         | 14  | video 2     |
| 15  | ground         | 16  | video 3     |
| 17  | ground         | 18  | unknown 2   |
| 19  | ground         | 20  | unknown 3   |

## Video Adapter Dongle

The VMP has a 20-pin IDC header, which connects by a ribbon cable to a DB25 male connector mounted in the rear cover of the Portable. This presumably connects to a dongle which provides the last step of signal conditioning and the correct pinout for the monitor in question.

We do not have the original dongle, however based on experimentation we have worked out its probable functions:

* The dongle provides a loopback between the +5V power supply and the power input to the oscillators. This is presumably to reduce power consumption when the dongle isn’t connected and the card is not in use.
* The DB25 contains a sense line to the GAL which is responsible for programming the CPLD. On the dongle, this line is pulled up to +5V, allowing the GAL to be programmed and the card to become operational only when the dongle is connected at boot.
* The dongle may provide level-shifting for driving analog monitors (e.g. VGA, Apple 13”). For TTL monitors a different dongle might have been used which passed through the video signal directly.

The pinout of the DB25 male is as follows:

| pin | description    | pin | description |
| --- | -------------- | --- | ----------- |
| 1   | oscillator +5V | 14  | +5V out     |
| 2   | ground         | 15  | sense input |
| 3   | ground         | 16  | HSYNC       |
| 4   | ground         | 17  | VSYNC       |
| 5   | ground         | 18  | unknown 1   |
| 6   | ground         | 19  | video 1     |
| 7   | ground         | 20  | video 2     |
| 8   | ground         | 21  | video 3     |
| 9   | ground         | 22  | unknown 2   |
| 10  | ground         | 23  | unknown 3   |
| 11  | n/c            | 24  | n/c         |
| 12  | n/c            | 25  | n/c         |
| 13  | n/c            |     |             |

