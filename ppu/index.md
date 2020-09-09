[<< Back to Main Page](../)

# The PPU

- [An Introduction](#an-introduction)
- [A Word of Warning](#a-word-of-warning)
- [The Basics](#the-basics)
  * [The Screen](#the-screen)
  * [The Concept of Tiles](#the-concept-of-tiles)
    + [Pixel Data](#pixel-data)
  * [Display Layers](#display-layers)
    + [The Background](#the-background)
    + [The Window](#the-window)

## An Introduction

The PPU (which stands for Picture Processing Unit) is the part of the Gameboy that’s responsible for everything you see on screen. It is the second most integral part of the whole machine right after the CPU, however, in terms of complexity it beats the CPU by far. The PPU is a lot more tricky and less unambiguously documented, which is why I’m hoping to provide an easy and quick to understand summary of everything here.

## A Word of Warning

I want to be clear about one thing: This documentation is most likely not 100% fully accurate. It’s a combination of information from various sources as well as experiences I’ve personally made while developing my emulator. If you find anything off, please do contact me personally on Discord (Optix™#1337) or create a PR.

**Also,** the current version of this documentation focuses only on DMG emulation, no CGB or SGB features are considered.

## The Basics

### The Screen

The screen of the original DMG Gameboy is a 160x144 pixel LCD which can display up to 4 different shades of gray. These appear green on the original DMG, but many emulators support custom color palettes to allow the user to manually configure which colors should be displayed.

### The Concept of Tiles

While the screen has a resolution of 160x144 pixels, pixels cannot be "written to" directly. The Gameboy can only display 8x8 pixel tiles. These are more or less "fixed 8x8 pixel textures" which can be placed on the screen in a set grid. Some exceptions apply to this, but these will be explained later on.

![8x8_tile](./8x8_tile.png)

#### Pixel Data

 Since the original DMG Gameboy only supports a palette of 4 different colors, 2 bits are needed to store color data for a single pixel. Hence, the Gameboy stores graphics data in a format commonly referred to as "2BPP", which stands for "2 bits per pixel".

In the Gameboy's 2BPP format, 2 bytes make up a row of 8 pixels. Each bit of the first byte is combined with the bit at the same position of the second byte to calculate the color number, as shown here, using `0xC3 0xA5` as example data:

![gb_2bpp](./gb_2bpp.png)

### Display Layers

The Gameboy display effectively uses three separate layers to display graphics on - the Background, the Window and Sprites.

#### The Background

The Background is a 32x32 tile grid (=256x256 pixels) in which Tiles can be placed. However, the Gameboy can only show a 20x18 tile (=160x144 pixels) section of the background at any given time. This section will be referred to as the "viewport" here. Which section is shown can be determined using the SCX and SCY registers (explained later on), which can be set by the program. The values in these registers determine how many pixels the viewport is offset from the left and the top border of the background respectively. If the viewport exceeds the border of the background it wraps back around to the left/top respectively.

![grid_bg](./grid_bg.png)

#### The Window

The Window is the same as the Background in that it is another 32x32 tile grid which Tiles can be placed on. However, it can be seen as a sort of "overlay" over the background, the position of which can be determined using WX and WY registers, which will also be explained later on.

![grid_win](./grid_win.png)