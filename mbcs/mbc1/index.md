[<< Back to Last Page](../)

# MBC1

The MBC1 is a very common MBC which supports up to 128 ROM banks (2MB) as well as up to 4 RAM banks (32KB).

## MBC Capabilities

The MBC needs to keep track of the following values:

* **ROM Bank Number:** Unsigned 5-bit number which (partly) determines which 16KB block of ROM is mapped to the memory region $4000 - $7FFF.
* **RAM Bank Number:** Unsigned 2-bit number which determines which 8KB block of external RAM is mapped to the memory region $A000 - $BFFF. It is also combined with the ROM Bank Number for ROM banking under certain conditions. [(See 'Modes and Banking')](#modes-and-banking)
* **Mode Flag:** A simple "zero or one" flag which determines the "Mode" the MBC is in. (Used in ROM / RAM banking)

If the MBC has a battery (MBC type identifier $03), the state of external RAM should be stored to a file when the emulator closes and reloaded once it starts the same game again.

## Memory Writes

### $0000 - $1FFF (Enable RAM)

Writing to a memory address between $0000 and $1FFF enables external RAM if the lower 4 bits of the written value are `0xA`. If they are not, external RAM will be disabled.

### $2000 - $3FFF (ROM Bank)

Writing to a memory address between $2000 and $3FFF will set the ROM Bank Number. However, a certain amount of bits (starting with the highest bit) is ignored depending on the total ROM size:

| **ROM Size**     | **Bitmask** |
| ---------------- | ----------- |
| 128 Banks (2MB)  | 0b00011111  |
| 64 Banks (1MB)   | 0b00011111  |
| 32 Banks (512KB) | 0b00011111  |
| 16 Banks (256KB) | 0b00001111  |
| 8 Banks (128KB)  | 0b00000111  |
| 4 Banks (64KB)   | 0b00000011  |
| 2 Banks (32KB)   | 0b00000001  |

The written value is ANDed with the bitmask according to the table above. If the written value (NOT the AND result) is zero, the ROM bank value is set to 1, otherwise it is set to the result of the AND operation. (In the case of a 32KB ROM, the ROM bank value will always be set to 1)

### $4000 - $5FFF (RAM Bank)

Writing to a memory address between $4000 and $5FFF will set the 2 bits of the RAM bank number to the lowest 2 bits of the written value. This value is, under certain conditions, also combined with the ROM Bank Number for ROM banking. [(See 'Modes and Banking')](#modes-and-banking)

### $6000 - $7FFF (Mode Select)

Writing to a memory address between $6000 and $7FFF will set the Mode Flag to the lowest bit of the written value.

### $A000 - $BFFF (External RAM)

Writing to a memory address between $A000 and $BFFF will write to external RAM if it is enabled. If it is not, the write is ignored.

The address in external RAM to which the value is written to depends on the total RAM size as well as the RAM bank number. If the total RAM size is either 2KB or 8KB, the address is simply `(ADDRESS - 0xA000) mod RAM_SIZE`.

If the total RAM size is 32KB (4 RAM banks) and the Mode Flag is set to 1, the address is calculated as follows: `0x2000 * RAM_BANK_NUMBER + (ADDRESS - 0xA000)`. If the Mode Flag is set to 0, the address is `ADDRESS - 0xA000`.

## Memory Reads

### $0000 - $3FFF (ROM Bank 0)

If the Mode Flag is zero, reading from a memory address between $0000 and $3FFF returns the byte at the corresponding address of the ROM file.

If the Mode Flag is one, a bank number which I will refer to as the "Zero Bank Number" is determined. The way this bank number is determined is explained [here.](#zero-bank) Reading from a memory address between $0000 and $3FFF under these conditions returns the byte at `0x4000 * ZERO_BANK_NUMBER + ADDRESS` of the ROM file.

### $4000 - $7FFF (ROM Bank 01-7F)

Reading from a memory address between $4000 and $7FFF, no matter what state the Mode Flag is in, always returns the byte at `0x4000 * HIGH_BANK_NUMBER + (ADDRESS - 0x4000)` of the ROM file. The way the "High Bank Number" is determined is explained [here.](#high-bank)

### $A000 - $BFFF (RAM Bank 0-3)

Reading from a memory address between $A000 and $BFFF returns a value from external RAM if it is enabled. If it is not, 0xFF is read.

The address in external RAM which is read from depends on the total RAM size as well as the RAM bank number. If the total RAM size is either 2KB or 8KB, the address is simply `(ADDRESS - 0xA000) mod RAM_SIZE`.

If the total RAM size is 32KB (4 RAM banks) and the Mode Flag is set to 1, the address is calculated as follows: `0x2000 * RAM_BANK_NUMBER + (ADDRESS - 0xA000)`. If the Mode Flag is set to 0 the address is `ADDRESS - 0xA000`.

## Modes and Banking

The MBC1 features a "Mode Flag" which can be modified by writing to a memory address between $6000 and $7FFF. This Flag affects how the process of both ROM as well as RAM banking works.

### Zero Bank

A "Zero Bank Number" is the bank number which is used to calculate the address in the ROM file to use when reading from memory addresses between $0000 and $3FFF while the Mode Flag is set to 1. This number depends on both the RAM bank number as well as the total ROM size.

#### 32 Banks or less (< 1MB)

If the ROM is less than 1MB in size (32 banks or less) the Zero Bank is always 0. Reads from memory addresses between $0000 and $3FFF will always return the bytes at the same address in the ROM file.

#### 64 Banks (1MB)

If the ROM is 1MB in size (64 banks) the Zero Bank Number is determined by the lower bit of the 2-bit RAM bank number. It takes the place of bit 5 of the Zero Bank number, allowing for the Zero Bank Number to be either 0x00 (`0b00000000`) or 0x20 (`0b00100000`).

**Exception:** Multi-Cart ROMs (often referred to as MBC1M) function slightly differently. The Zero Bank Number is determined by both RAM bank number bits taking place of bits 4 and 5 of the Zero Bank number, allowing for it to be one of the following: 0x00 (`0b00000000`), 0x10 (`0b00010000`), 0x20 (`0b00100000`) or 0x30 (`0b00110000`).

#### 128 Banks (2MB)

If the ROM is 2MB in size (128 banks) the Zero Bank Number is determined by the entire 2-bit RAM bank number. The two bits take the place of bits 5 and 6 of the Zero Bank number respectively, allowing for the Zero Bank Number to be one of the following: 0x00 (`0b00000000`), 0x20 (`0b00100000`), 0x40 (`0b01000000`) or 0x60 (`0b01100000`).

### High Bank

A "High Bank Number" is the bank number which is used to calculate the address in the ROM file to use when reading from memory addresses between $4000 and $7FFF. This number depends on the ROM bank number and, under certain conditions, also both the total ROM size as well as the RAM bank number.

#### 32 Banks or less (< 1MB)

If the total ROM size is less than 1MB (32 banks or less) the High Bank Number is simply the ROM Bank number ANDed with the bitmap of the corresponding ROM size. [(Ref. Memory Writes to $2000-$3FFF)](2000-3fff-rom bank)

#### 64 Banks (1MB)

If the ROM is 1MB in size (64 banks) the High Bank number is, at first, calculated the same way as the 32-Bank number. However, the lower bit of the 2-bit RAM bank number takes the place of bit 5.

#### 128 Banks (2MB)

If the ROM is 2MB in size (128 banks) the High Bank number is calculated the same way as the 64-Bank number, except both bit of the RAM bank number are used for bits 5 and 6 of the end result respectively.