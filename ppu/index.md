[<< Back to Main Page](../)

# The PPU

## An Introduction

The PPU (which stands for Picture Processing Unit) is the part of the GameBoy that's responsible for everything you see on screen and more and possibly the second most integral part of the whole machine right after the CPU. While it is technically less complex than the CPU, it's a lot more tricky and less unambiguously documented, which is why I'm hoping to provide an easy and quick to understand summary of everything here.

## A Word of Warning

I want to be clear about one thing: This documentation is most likely not 100% fully accurate. It's a combination of information from various sources as well as experiences I've personally made while developing my emulator. If you find anything off, please do contact me personally on Discord (Optix™#1337) or create a PR.

## The Basics

### The Screen

As mentioned before, one of the tasks of the PPU is to act as the interface to the video output for the whole machine. The screen is a 160x144 pixel LCD which supports up to 4 different shades of gray (which look green on the original DMG).

### What's Shown on Screen

Effectively, the GameBoy has three "layers" which it can draw on - the Background, the Window and Object Sprites, all of which will be elaborated on later on in the documentation. One thing that is true for all layers is that the images are based on 8x8 pixel tiles, which are stored in memory as 16 bytes each. How exactly the image data is stored will be explained later on as well.

### The Tech Behind It

Before we get into the technology let's define an important term first - a "scanline". A scanline is a single row of 160 pixels which are drawn to the screen, so any GameBoy-rendered image has a total of 144 scanlines.

Now that that's all cleared up, let's get to the technical part. Many emulators implement image rendering on a scanline-basis, which means that one whole scanline is drawn at once. This works perfectly fine for most games, but there are some exceptions which use special effects that can only be achieved with what's known as a "Pixel FIFO" implementation.

## PPU Modes

The PPU operates in 4 different modes, the timing of which is visualized in the image below (taken from the GameBoy Pandocs). The details of each mode will be explained shortly.

![img](https://github.com/corybsa/pandocs/raw/develop/content/imgs/game-boy-lcd-refresh-diagram.png)

### 154 Scanlines?

According to the diagram above, during the rendering of one frame the PPU handles a total of 154 scanlines, even though the LCD display only has 144 rows. This is due to what's known as VBlank Mode, which adds 10 pseudo-scanlines to the bottom of the frame where nothing is actually rendered and the PPU is effectively paused. More information on that shortly.

### Mode 2 - OAM Search

The first mode the PPU is in when starting a scanline (that isn't VBlank) is the OAM Search mode. It takes a total of 80 T-cycles and is responsible for scanning memory for sprites that should be rendered on the scanline the PPU is currently rendering.

During OAM Search the VRAM Sprite Attribute Table is scanned, the scan for each entry taking 2 T-cycles. For every Sprite a few conditions are checked and, if all apply, the Sprite is pushed to a separate OAM Buffer (which has nothing to do with the Pixel FIFO). The conditions are as follows:

* Sprite X-Position must be greater than 0
* LY + 16 must be greater than or equal to Sprite Y-Position
* LY + 16 must be less than Sprite Y-Position + Sprite Height
* The amount of sprites already stored in OAM Buffer must be less than 10

**Note:** During Mode 2 the CPU should not be able to access the OAM part of memory ($FE00 - $FE9F). Any attempts at writing to it should be ignored and reading from it should return 0xFF.

### Mode 3 - Drawing

Mode 3 is the "main" mode of the PPU, this is where the magic happens. Its length is variable and changes depending on how many sprites are to be drawn on the scanline, whether or not the window should be drawn on the scanline and many more factors, all of which will also be explained later on.

**Note:** During this mode the CPU shouldn't be able to access either OAM ($FE00 - $FE9F) or VRAM ($8000 - $9FFF). Like in Mode 2, any attempts at writing to it should be ignored and reading from it should return 0xFF.

### Mode 0 - HBlank

This mode takes up the remainder of every scanline after drawing is done, and always lasts until the scanline the PPU is currently rendering has been worked on for 456 T-Cycles. During this mode all memory is accessible by the CPU, which is why some games use this time segment to modify data in OAM or VRAM.

### Mode 1 - VBlank

VBlank mode takes up all scanlines from 144 to 153 (zero-based) and effectively acts as a "PPU pause" the same way HBlank does. It is however considerably longer and allows for the game to make larger changes to OAM and VRAM.

## The Pixel FIFO

The Pixel FIFO, in most general terms, is a FIFO (First In First Out) Buffer which stores pixels and pushes them out onto the LCD one by one. The FIFO itself however is only the "storage" part of that system. The part that loads pixel data into the FIFO is known as the "Fetcher".

### FIFO Operation

Below is a simplified diagram of how the Pixel FIFO operates. The Fetcher gets pixel data from memory and pushes it into the FIFO, while the PPU is shifting out pixels to the LCD on the other side. This operation is repeated scanline by scanline until a full image is rendered, then it restarts. (Not right away, more details later)

![pixelfifo_basic](./pixelfifo_basic.png)

#### What's a Pixel?

What's shown as a "Pixel" in the image above isn't just data that shows what color to display on screen, one pixel has to store multiple attributes:

* The Color Number (2 bits => value from 0 to 3)
* The Color Palette (Only really necessary for pixels that belong to sprites)
* Whether or not the pixel belongs to a sprite

#### Pixel Data

As mentioned before, GameBoy graphics work with 8x8 pixel tiles which are stored as 16 bytes. Each 2 bytes make up a row of 8 pixels, which is why 2 * 8 = 16 bytes make up a whole 8x8 tile.

A great explanation of how 2 bytes make up a row of 8 pixels can be found [here](https://www.huderlem.com/demos/gameboy2bpp.html). (TODO: Write own description)

#### Background Rendering

The simplest utilization of the Pixel FIFO is basic background rendering. No sprites, no window, just background. Here we'll get to know the Fetcher a little more and look into how it operates in detail.

##### Background Pixel Fetching

Once the OAM Scan is done, background pixel fetching begins. And, usually, this process keeps repeating over and over until the scanline is fully rendered. Fetching 8 pixels and pushing them to Pixel FIFO takes a total of 8 T-cycles.

The fetcher internally keeps track of which tile it is fetching (I will refer to this as LX, to stay consistent with the generally agreed upon naming of the LY register). For each tile on the scanline the following operations occur:

* **Cycle 1-2:** The fetcher reads the tile number from memory.
* **Cycle 3-4:** The fetcher reads the lower 8 bits of the 8-pixel-row data.
* **Cycle 5-6:** The fetcher reads the upper 8 bits of the 8-pixel-row data.
* **Cycle 7-8:** The fetcher pushes the 8 pixels onto Pixel FIFO.

##### Combining Fetching and Drawing

Now that we know how to fetch background pixels we can start combining them with the drawing logic.

The first 6 T-Cycles of the scanline nothing is drawn, as the Pixel FIFO is still empty and the fetcher is going through its first steps. On the 7th Cycle the fetcher pushes the first 8 pixels to the FIFO, and the first pixel is shifted out onto the LCD immediately in the same cycle after being pushed to FIFO.

On the 8th cycle the fetcher resets and another pixel is shifted out onto LCD, leaving 6 more pixels in FIFO. These are shifted out every T-cycle, while the fetcher runs through its first 6 fetching cycles. Then again, on the 12th cycle total, the fetcher pushes 8 more pixels to FIFO, one of which is again shifted to LCD in the same cycle. This cycle continues over and over until the PPU shifts out the 160th pixel.

Once it has reached the 160th pixel the PPU resets everything Pixel FIFO-related. It clears anything that's left in FIFO and fully resets the fetcher to its initial state at the beginning of the scanline in order to get it ready for the next scanline.

##### The first-tile oddity

Doing the math, the drawing mode, when only drawing background tiles, should take 166 T-cycles per scanline. However, the minimum amount of cycles needed for Mode 3 is 172. Why?

This is due to an oddity with the background fetcher. It starts operating as usual at the start of the scanline, reading the tile number, the lower and the upper 8 bits of tile data. However, it discards all data it fetched and restarts itself without pushing any data to the FIFO. This adds the 6 T-cycles to the beginning of Mode 3, adding it up to 172 cycles.

##### Timing Visualization

![pixelfifo_bg](./pixelfifo_bg.png)
