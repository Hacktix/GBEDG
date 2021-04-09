[<< Back to Last Page](../)

# Memory Bank Controllers (MBCs)

Memory Bank Controllers (most commonly abbreviated to MBCs) are pieces of hardware that are built into the Game Cartridge, not the Gameboy itself. Because storage was very expensive in the era of the Gameboy games had to be small - yet the 32KB limit that the unmodified Gameboy design intended wasn't enough for most games. Additionally, a lot of games would need external hardware such as Real Time Clocks, Rumble or Battery-powered external RAM to store save files. All of these limitations were more or less bypassed by MBCs - additional hardware on the cartridge.

## Documented MBCs

* [MBC1](mbc1)
* [MBC2](mbc2)
* [MBC3](mbc3)
* [MBC5](mbc5)

## Table of Contents

- [How to detect a ROMs MBC](#how-to-detect-a-roms-mbc)
- [Additional Hardware](#additional-hardware)
  * [None / Plain ROM](#none--plain-rom)
  * [(Battery-Buffered) RAM](#battery-buffered-ram)
  * [Real Time Clock (RTC)](#real-time-clock-rtc)
  * [Rumble](#rumble)
  * [Accelerometer](#accelerometer)
- [Storage Sizes](#storage-sizes)
  * [ROM Size](#rom-size)
  * [RAM Size](#ram-size)
- [How to handle incorrect header data](#how-to-handle-incorrect-header-data)

## How to detect a ROMs MBC

In the world of emulators we're not dealing with actual hardware - all we have are ROM files. These provide the data stored on the cartridge, but not the functionality of it - that part has to be emulated.

Gameboy ROMs feature a so-called "Cartridge Header" section in memory which contains all sorts of metadata about the game stored on the cartridge. For the detection of the MBC type the only relevant byte in the ROM file is located at $0147. This byte is an identifier for the individual MBC type as well as the additional hardware on the cartridge, such as external RAM, Real Time Clocks or Rumble.

| **Code** | **MBC Type**  | **Additional Hardware**                       |
| -------- | ------------- | --------------------------------------------- |
| 0x00     | None          | None, plain ROM                               |
| 0x01     | MBC1          | None                                          |
| 0x02     | MBC1          | RAM                                           |
| 0x03     | MBC1          | Battery-Buffered RAM                          |
| 0x05     | MBC2          | None                                          |
| 0x06     | MBC2          | Battery-Buffered RAM                          |
| 0x08     | None          | RAM                                           |
| 0x09     | None          | Battery-Buffered RAM                          |
| 0x0B     | MMM01         | None                                          |
| 0x0C     | MMM01         | RAM                                           |
| 0x0D     | MMM01         | Battery-Buffered RAM                          |
| 0x0F     | MBC3          | Real Time Clock                               |
| 0x10     | MBC3          | Real Time Clock + Battery-Buffered RAM        |
| 0x11     | MBC3          | None                                          |
| 0x12     | MBC3          | RAM                                           |
| 0x13     | MBC3          | Battery-Buffered RAM                          |
| 0x19     | MBC5          | None                                          |
| 0x1A     | MBC5          | RAM                                           |
| 0x1B     | MBC5          | Battery-Buffered RAM                          |
| 0x1C     | MBC5          | Rumble                                        |
| 0x1D     | MBC5          | Rumble + RAM                                  |
| 0x1E     | MBC5          | Rumble + Battery-Buffered RAM                 |
| 0x20     | MBC6          | None                                          |
| 0x22     | MBC7          | Accelerometer + Rumble + Battery-Buffered RAM |
| 0xFC     | POCKET CAMERA | ?                                             |
| 0xFD     | BANDAI TAMA5  | ?                                             |
| 0xFE     | HuC3          | Real Time Clock + Buzzer **(?)**              |
| 0xFF     | HuC1          | Battery-Buffered RAM                          |

## Additional Hardware

The list above contains a lot of terms that may seem confusing or very broadly defined - so here's a short summary of what each term means:

### None / Plain ROM

In the case of code `0x00` this means that the ROM file is just that - Read Only Memory. No tricks involved. Other cartridges which do have an assigned MBC type have *only* the given MBC chip on them and nothing more.

### (Battery-Buffered) RAM

This means that the cartridge has an additional RAM chip inside the cartridge in order to, well, *have more RAM.* As RAM is inherently volatile (meaning the data is lost when the power cuts out) it may be constantly supplied with power by an additional in-cartridge battery. This is called Battery-Buffering and allows for things such as savegames to exist, as the data in Battery-Buffered RAM is preserved even when turning off the Gameboy (or Emulator in this case). Most emulators implement this by using a save file which the RAM data is written to just before the emulator is closed.

### Real Time Clock (RTC)

A Real Time Clock (often abbreviated to RTC) is dedicated hardware for counting real-time seconds, minutes, hours and days. The RTC that comes with the MBC3 is inherently Battery-Buffered and keeps count of the time even when the Gameboy / Emulator is turned off. Games like Pokemon Silver and Pokemon Gold use these to keep track of the real time to display in game.

### Rumble

A small amount of Gameboy games were built with an additional Rumble Pak attached, which would be able to vibrate through specific functions in the games code. **(TODO: Search for Documentation)**

### Accelerometer

The accelerometer is an MBC7 specific hardware addition which comes packaged with a Rumble Pak (explained above). It allows games to detect the angle at which the physical Gameboy console is being held - a sort of primitive motion control system.

## Storage Sizes

Now we know how to detect the MBC type of a ROM and its hardware, but we still don't know the storage size of the ROM or external RAM if there is any. However, this is just as easy as determining the MBC type.

### ROM Size

An identifier representing the ROM size is stored in the ROM file at address $0148:

| **Code** | **ROM Size**            | **ROM Banks**                   |
| -------- | ----------------------- | ------------------------------- |
| 0x00     | 32KB (32,768 Bytes)     | None (technically all one Bank) |
| 0x01     | 64KB (65,536 Bytes)     | 4 Banks                         |
| 0x02     | 128KB (131,072 Bytes)   | 8 Banks                         |
| 0x03     | 256KB (262,144 Bytes)   | 16 Banks                        |
| 0x04     | 512KB (524,288 Bytes)   | 32 Banks                        |
| 0x05     | 1MB (1,048,576 Bytes)   | 64 Banks                        |
| 0x06     | 2MB (2,097,152 Bytes)   | 128 Banks                       |
| 0x07     | 4MB (4,194,304 Bytes)   | 256 Banks                       |
| 0x08     | 8MB (8,388,608 Bytes)   | 512 Banks                       |
| 0x52     | 1.1MB (1,179,648 Bytes) | 72 Banks                        |
| 0x53     | 1.2MB (1,310,720 Bytes) | 80 Banks                        |
| 0x54     | 1.5MB (1,572,864 Bytes) | 96 Banks                        |

**Wait, what are ROM banks?**

ROM banks are effectively 16KB "batches" of memory which can be mapped to a certain addressing space individually. With MBC1 for example the memory region $4000 to $7FFF can be mapped to (almost) any 16KB block of the ROM.

### RAM Size

The header also contains an identifier representing the size of however much external RAM there is at address $0149:

| **Code** | **RAM Size**                                     |
| -------- | ------------------------------------------------ |
| 0x00     | None                                             |
| 0x01     | 2KB (2,048 Bytes)                                |
| 0x02     | 8KB (8,192 Bytes)                                |
| 0x03     | 32KB (32,768 Bytes split up into 4 RAM banks)    |
| 0x04     | 128KB (131,072 Bytes split up into 16 RAM banks) |
| 0x05     | 64KB (65,536 Bytes split up into 8 RAM banks)    |

## How to handle incorrect header data

1) Except for some specific edge cases, you can't

2) You don't really need to

Any Nintendo-licensed game contains proper header data, and even a lot of bootleg games do. Even if the data is wrong - that's a corrupt ROM file, not an issue with the emulator.