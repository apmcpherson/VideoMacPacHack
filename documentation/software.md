# Video Mac Pac software

The *Video Mac Pac 4.1* control panel (CDEV) contains the drivers for the CPLD and several options for controlling the card. It allows a choice of resolutions, which take effect on power cycle -- not necessarily on reset since the CPLD needs to be powered down. 

The external screen can be configured to be on the left or the right of the Portable screen, and the menu bar can be moved between screens. The Portable screen can also be disabled. These changes also take effect at restart.

When the Portable sleeps, the CPLD settings are lost and the external screen does not reappear on wake. The machine needs to be rebooted for the CPLD code to be reloaded.

## Resolutions

The control panel offers a choice of the following resolutions (all at 1-bit monochrome). The VSYNC and HSYNC frequencies of each resolution were measured, and it was determined which of the three oscillator cans each resolution was derived from:

| Resolution | Name           | Frequencies         | Oscillator | Notes     |
| ---------- | -------------- | ------------------- | ---------- | --------- |
| 640x872    | Apple Portrait | 68.8kHz H, 74.5Hz V | 57.2832MHz |           |
| 640x872    | Sunkyung FPD   | 68.8kHz H, 74.5Hz V	| 57.2832MHz | no video  |
| 640x872    | Samsung FPD	  | n/a                 | n/a        | see below |
| 640x480    | Apple M/C      | 39.9kHz? H, 76Hz? V	| 36MHz      | see below |
| 640x480    | VGA            | 31.5kHz H, 60Hz V   | 50.35MHz   |           |
| 800x600    | Super VGA      | 35.1kHz H, 56Hz V   | 36MHz      |           |
| 528x348    | Mono TTL       | 18.6kHz H, 50Hz V   | 50.35MHz   | no video  |
| 272x208    | TV/VCR         | n/a                 | n/a        | untested  |

The `Sunkyung FPD` and `TV/VCR` modes appear to use a different pin for video output, so they do not directly work with the dongle we have developed. Mono TTL could not be tested as it was below the scan range of the monitor for testing, but may also use a different video pin.

The `Samsung FPD` mode does not work because its corresponding `DS#1` resource is corrupt, missing a few bytes of the CPLD code.

The `Apple M/C` mode produces the wrong frequenceis for the Apple Monochrome Monitor. This mode may expect a different oscillator to be installed in the 36MHz socket, e.g. 31.5MHz, to produce a 66.7Hz refresh as expected.

## CDEV Resources

The Video Mac Pac control panel (CDEV) appears to be derived from the Lapis DisplayServer CDEV, which is used in several Lapis video cards for the Macintosh SE and later machines. 

Some of the important resources in the CDEV include:

* `Glue`: Holds 68k code implementing the display driver and hardware communication.
* `DS#1`: There are several such resources in the CDEV. Each one corresponds to a display resolution.
	* Offset `$92`: Holds the length of the CPLD code in bytes. The value in each case is `$8BC`: 2236 bytes, or 17888 bits. The XC2018 has a program space of 17878 bits.
	* Offset `$94`: CPLD bit stream begins
* `UVDS`: Holds the information on the current configuration ID (referring to an ID of a `DS#1` resource) and the settings for display arrangement, menu bar, and whether the internal screen is used.

For comparison: `DS#1` resources can also be found in the Lapis Dual Screen Software INIT. The DisplayServer PDS/30 CDEV contains `DS#4` resources, and the DisplayServer II-DPD CDEV contains `DS#6` resources, which appear to operate on similar principles. In principle, it might be possible to find CPLD code to support other resolutions from another CDEV or to reverse engineer it with contemporary Xilinx development tools.

## Memory-Mapped Addressing

The Macintosh Portable is a 68000 machine with 24-bit addressing. It reserves address range `$000000` to `$8FFFFF` for RAM (for a maximum of 9MB total).

On the Video Mac Pac, I/O is memory-mapped with the lowest region starting at `$720000`. This implies that the VMP will not function correctly if more than 7MB of RAM is installed in the Portable. (The largest factory configuration was 4MB (1MB onboard + 3MB expansion), but modern expansion cards are available providing up to 9MB total RAM.)

An analysis of the CDEV shows the following addresses in use: 

* `$720000`: VRAM base, addressable once CPLD code is loaded
	* Maximum length `$20000` (128 kB = max 1M pixels)
	* `$73FFEA` is used in the CDEV for a special checksum word, `BECK`, which is written and subsequently read to detect the presence of the card
	* `$73FFEE` is used in the CDEV to hold the ID of the current `DS#1` resource which contains information on the screen resolution
	* These two checks also serve to confirm that the CPLD has been programmed, since the VRAM is only accessible on the bus by way of the CPLD.
* `$740000`: unknown function
	* This address is referenced in various places in the CDEV code. For example, before the CPLD code is loaded by writing it out to `$740200`, the CDEV writes `$FF` to address `$740000`. However, it is not clear what the function of this write is or whether the address has any specific meaning to the hardware.
* `$740200`: `WRT` pin on CPLD
	* Any memory access to this address causes the GAL to bring the `WRT` pin on the CPLD low. Because data line `D0` is attached to the `DIN` pin of the CPLD, the practical effect is that each write to `$740200` shifts in a single bit of the CPLD program. 
	
The programming procedure for the CPLD thus consists of the following for each byte of the `DS#1` resource:

* Write byte to `$740200`
* Shift byte 1 bit right
* Write again
* Repeat the above 8 times in total, then move to the next byte



