# Video Adapter Dongle

An adapter board is necessary to connect from the DB25 male on the back of the Portable to the monitor. The original dongle for the Video Mac Pac was lost, so this reconstruction is based on its apparent principles of operation and almost certain differs substantially from the original.

See the `hardware/VGA_adapter/` directory in this repository for the schematic and board layout of the adapter. 

## Detection Circuit

Connecting pins 14 and 15 on the DB25 (pins 2 and 4 on the 20-pin IDC) holds the sense line high and signals the presence of the dongle. 

In the schematic, a 100 ohm resistor is used between these as a precaution in case of incorrect insertion of the dongle, but this could probably be replaced with a simple short.

## Oscillator Power

Connecting pins 1 and 14 on the DB25 (pins 1 and 2 on the 20-pin IDC) provides +5V power to the oscillators which are needed to generate the video pixel clock.

## Buffering and Level Shifting

All signals from the card are 5V TTL. VGA signals need to be 0.7V or 1.0V maximum, depending on the standard. Level shifting is therefore required for the video signals. 

The sync signals remain at TTL levels and are passed through unchanged to the VGA connector.

### 3.3V regulator

While a simple voltage divider could reduce the 5V TTL video signal to the correct level, we find that the 5V line on the Portable fluctuates under load, which results in brightness variations and sometimes noise on the picture.

Instead, a 3.3V regulator is used to generate a stable reference voltage for the video signals. Each of the three video signals is buffered by a `74LVC245` level shifter, which produces 3.3V outputs. These are in turn divided down by a resistive voltage divider to meet VGA level and impedance requirements.

### Impedance

VGA expects a signal of up to 0.7V (1.0V in some variations), and an impedance of 75 ohms. The `74LVC245` has a rated output impedance of 15 ohms per buffer. We therefore use 133 ohms in series (closest available value to 135 ohms) and 150 ohms in parallel to produce a raw (unloaded) output of 3.3V/2 = 1.65V, with a source impedance of 150/2 = 75 ohms. When the monitor is connected, the 75 ohm load impedance reduces the video signal to 1.65V/2 = .825V.

For comparison: examining the video adapters for the Lapis DisplayServer cards shows that they used 150 ohms in series and 150 ohms in parallel from a 5V source to generate the video signal.

### Alternative approach

Three separate logic signals are used for the R, G, and B lines, which ultimately derive from three different pins on the CPLD. Slight timing shifts between the pins result in messy vertical edges in the image, with colour tints to the left or right of the line. 

In our later investigations, we found that the `74HC244` had too long and too variable a propagation delay and created noticeable clarity problems. The `74F244`, used on the original VideoMacPac design, does not show this problem. However, since the image is monochrome anyway, a relatively foolproof alternative is to drive all three buffers on the 74LVC245 with the same video signal.

## PCB Design

A 2-layer PCB was used with surface mount ICs and resistors for compactness. The DB25 and VGA connectors are on opposite ends of the board. 

The board could be enclosed in a plastic housing, or a version could be developed that goes directly from the 20-pin IDC to a VGA port. If this adapter were connected all the time, a switch could be used to toggle the sensing function that activates the card.


