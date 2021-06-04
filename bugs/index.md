[<< Back to Last Page](../)

# Common Bugs & Issues

The following is a list of common bugs that occur when starting up games on in-development emulators and potential sources these issues could be caused by. If you encounter any bugs while developing your own emulator that have not been listed on this page, feel free to contact me on Discord (Optix™#3264) and I'll add it.

## Tetris

### Flickering Copyright Notice

A very common issue for newly developed emulators is flickering on the following copyright notice screen:

![Tetris Copyright Screen](./tetris_title.png)

This is most likely caused by the I/O register at memory address `$FF00` (which is the Joypad Register) returning zero when read from. This is due to the fact that the bit values of the Joypad registers are inverted - 0 means "pressed" while 1 means "not pressed". Tetris has a "reset" button combo which resets the game fully to the copyright notice screen if A+B+START+SELECT are all pressed. If reading from memory address `$FF00` returns zero, the game assumes all these buttons are pressed and resets, causing the flickering.

## The Legend of Zelda: Link's Awakening

### Broken Item Bar

When starting The Legend of Zelda: Link's Awakening, the bar at the bottom of the screen, in which the selected items should be displayed, may not be visible. Instead, something like this can be seen:

![zelda_links_awakening_window](./zelda_links_awakening_window.png)

This is most likely a problem with handling values of the WX register (`$FF4B`) that are below 7. The value of the WX register is often defined as "Window X Position plus 7", however, Link's Awakening sets the value of WX to 6, which new emulators may incorrectly evaluate to 255 by subtracting 7 while handling the value as an unsigned 8 bit integer. Instead, signed values should be used, as WX values below 7 should shift the window off the screen to the left.

## Pokémon Red/Blue Version

### Black Box on Title Screen

Instead of a proper Pokemon, the following black box may appear on the title screen of both the Red and Blue Version of the Pokémon Games:

![pokemon_black_box](./pokemon_black_box.png)

This is most likely an issue with the MBC implementations. Specifically, this occurs when reading from the SRAM section ($A000-$BFFF) returns `$FF`.
