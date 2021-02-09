[<< Back to Last Page](../)

# Hardware Overview

The following is a general overview of hardware present in the DMG Gameboy. More detailed explanations of each component can be viewed by clicking on the header of the component.

## [CPU](../cpu)

The Gameboy uses an 8-bit Sharp LR35902 processor running at 4.194304MHz. The CPU is a sort of "mix" of the Intel 8080 and Zilog Z80 processors, with some features missing and a few unique ones added. It features a total of 8 data registers, 6 of which can be combined into "register pairs" and used as 16-bit values or memory pointers. Additionally, the CPU has a 16-bit register for the Stack Pointer and the Program Counter respectively.

## Memory

The Gameboy uses 16-bit addressing, allowing for addressing from $0000 to $FFFF. This addressing space is mapped to ROM, WRAM, VRAM, Memory-mapped I/O and others, which are described in more detail in the memory section of this documentation.

## [Memory Bank Controllers (MBCs)](../mbcs)

MBCs are chips that are built into game cartridges in order to extend the basic functionality of the Gameboy hardware. This is mostly in regards to storage space, allowing for games larger than 32Kb, as well as external RAM which can be battery-powered in order to provide the ability to save games and highscores.

## [Timers](../timers)

The Gameboy features multiple I/O registers which are responsible for timer-related tasks. These are linked to the main CPU clock, however, there are some oddities in their implementation, which is why they are explained separately in this documentation.

## Display

The DMG Gameboy features a 160x144 pixel screen supporting four different shades of gray. It is refreshed at a rate of approximately 59.73Hz, which amounts to almost 60FPS.

## [PPU (Picture Processing Unit)](../ppu)

The PPU of the Gameboy is responsible for putting together and outputting graphics onto the LCD screen. It is treated as separate hardware from the CPU and is possibly the most challenging part of emulating the Gameboy in a hardware-accurate manner.