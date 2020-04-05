# VideoMacPacHack

The VideoMacPacHack is a clone of the Video Mac Pac, a PDS video card for the Macintosh Portable designed in 1991 by Computer Care Inc. The Video Mac Pac provides monochrome (1-bit) video on an external display, with several resolutions selectable by a control panel.

The VideoMacPacHack aims to replicate the Video Mac Pac using original parts where possible, with modern substitutions in  cases where original parts are hard to source (e.g. GAL, miniature relay), or where the original design was unknown (the video adapter board).

## Contents

* `hardware/VideoMacPac`: PCB design for the VideoMacPacHack, 4 layers, made in Kicad
* `hardware/VGA_adapter`: PCB design for the VGA breakout board, 2 layers, made in EAGLE
* `software`: Video Mac Pac Control Panel, needed to load the card firmware and select resolution
* `photos`: Photos of the original and clone card
* `documentation`: Technical information on the hardware and software

## System Requirements and Operation

The VideoMacPacHack fits the PDS slot of the Macintosh Portable. It has been successfully tested in both M5120 (non-backlit) and M5126 (backlit) Portables. Based on the address space used by the card, it will probably only work on Portables with less than 8MB of RAM installed.

The VideoMacPacHack (like the original Video Mac Pac) requires its associated control panel to run (see `software`). The external screen will become active when the control panel loads, provided the external video adapter is attached at boot. If the Portable is put to sleep, the external display will not work until reboot.

## Acknowledgments

The Video Mac Pac was created by Computer Care Inc. in 1991, apparently based on the Lapis DisplayServer.

The VideoMacPacHack was designed by [Andrew McPherson](https://github.com/apmcpherson) and [Tom Stepleton](https://github.com/stepleton). In the interest of historical preservation it is released into the public domain, with no warranty of any sort.