[<< Back to Last Page](../)

# Common Bugs & Issues

The following is a list of common bugs that occur when starting up games on in-development emulators and potential sources these issues could be caused by. If you encounter any bugs while developing your own emulator that have not been listed on this page, feel free to contact me on Discord (Optix™#3264) and I'll add it.

## Bomberman GB

### Unable to move left/right

In the Bomberman GB Games (both the Japanese and the US/Europe Versions), you may not be able to move left or right, while vertical movement works just as intended. This is most likely caused by incorrectly handling the case where both the Joypad as well as the Action Buttons are selected in the Joypad register. In this case, both the joypad buttons as well as the action buttons are mapped to their respective bits, and the bit is set to 0 if at least one of the two corresponding buttons is pressed.

## Dr. Mario

### Messed up Title Screen

Dr. Mario is, right after Tetris, one of the most common games to attempt to boot up early on in emulator development. However, while Tetris may look and function perfectly fine, Dr. Mario (as well as a few other games) may look like this:

![drmario_bgtiles](./drmario_bgtiles.png)

Seeing this exact image as the title screen of Dr. Mario is a common misunderstanding of the "8800 Addressing Mode" of the PPU, using the address `$8800` as the base address for tile data rather than the correct `$9000` address.

## Pokémon Red/Blue Version

### Black Box on Title Screen

Instead of a proper Pokemon, the following black box may appear on the title screen of both the Red and Blue Version of the Pokémon Games:

![pokemon_black_box](./pokemon_black_box.png)

This is most likely an issue with the MBC implementations. Specifically, this occurs when reading from the SRAM section ($A000-$BFFF) returns `$FF`.

## Tetris

### Flickering Copyright Notice

A very common issue for newly developed emulators is flickering on the following copyright notice screen:

![Tetris Copyright Screen](./tetris_title.png)

This is most likely caused by the I/O register at memory address `$FF00` (which is the Joypad Register) returning zero when read from. This is due to the fact that the bit values of the Joypad registers are inverted - 0 means "pressed" while 1 means "not pressed". Tetris has a "reset" button combo which resets the game fully to the copyright notice screen if A+B+START+SELECT are all pressed. If reading from memory address `$FF00` returns zero, the game assumes all these buttons are pressed and resets, causing the flickering.

### Non-random Blocks

Another fairly common beginner issue with Tetris is a lack of randomization for the block shapes that drop down ingame, such as the following:

![tetris_badrng](./tetris_badrng.png)

This is most likely due to an issue with the DIV register ($FF04), as it's used to "randomize" the shapes. In-development emulators may simply map this address to the value zero, resulting in the game always dropping 2x2 blocks. Other values may lead to other shapes being dropped.

### Crash when Starting Game or Demo

Tetris may crash in some way when attempting to start the game or waiting in the main menu for the demo to play. These crashes may look different depending on a few factors, ranging from a visually glitched main menu to a completely blank screen. However, this is most likely due to the emulator letting the game change values in ROM by writing to them. Specifically, Tetris writes a value to the memory address `$2000`, which should simply be ignored.

## The Legend of Zelda: Link's Awakening

### Broken Item Bar

When starting The Legend of Zelda: Link's Awakening, the bar at the bottom of the screen, in which the selected items should be displayed, may not be visible. Instead, something like this can be seen:

![zelda_links_awakening_window](./zelda_links_awakening_window.png)

This is most likely a problem with handling values of the WX register (`$FF4B`) that are below 7. The value of the WX register is often defined as "Window X Position plus 7", however, Link's Awakening sets the value of WX to 6, which new emulators may incorrectly evaluate to 255 by subtracting 7 while handling the value as an unsigned 8 bit integer. Instead, signed values should be used, as WX values below 7 should shift the window off the screen to the left.
