[<< Back to Last Page](../)

# MBC3

The MBC3 is a somewhat common MBC supporting up to 128 ROM banks (2MB) as well as up to 4 RAM banks 32KB) which can, optionally, be battery-buffered. Additionally, the MBC3 has a built-in Real Time Clock (RTC) which can also be battery buffered, keeping it ticking while the Gameboy itself is turned off.

## MBC Capabilities

The MBC needs to keep track of the following values:

* **ROM Bank Number:** Unsigned 7-bit number which determines which 16KB block of ROM is mapped to the memory region $4000 - $7FFF.
* **RAM Bank Number:** Unsigned 2-bit number which determines which 8KB block of external RAM is mapped to the memory region $A000 - $BFFF.

If the MBC has a battery (MBC type identifier $10 or $13), the state of external RAM should be stored to a file when the emulator closes and reloaded once it starts the same game again.

For a detailed description of the RTC, see [here.](#rtc)

## Memory Writes

### $0000 - $1FFF (Enable RAM / Timer Registers)

Writing to a memory address between $0000 and $1FFF enables external RAM as well as access to the RTC registers if the lower 4 bits of the written value are `0xA`. If they are not, external RAM will be disabled.

### $2000 - $3FFF (ROM Bank Low)

Writing to a memory address between $2000 and $3FFF will set the ROM Bank Value to the lower 7 bit of the written value. If the written value is 0, the ROM Bank Value is set to 1 instead.

### $4000 - $5FFF (RAM Bank / RTC Register Select)

Writing to a memory address between $4000 and $5FFF will either set the RAM bank value or select an RTC register depending on the written value.

If the written value is in the range $00 - $03, the RAM Bank Value is set to the written value and the External RAM is mapped to the corresponding memory region ($A000 - $BFFF).

If the written value is in the range $08 - $0C, the corresponding RTC register is mapped to the External RAM region ($A000 - $BFFF) instead. For details, check out the [RTC Section](#rtc) of this documentation.

### $6000 - $7FFF (RTC Data Latch)

A write of the value `$00` followed by another write with the value `$01` will "latch" (effectively copy) the current state of the RTC registers and make them accessible by using the RTC Register Select feature mentioned above. For details, check out the [RTC Section](#rtc) of this documentation.

### $A000 - $BFFF (External RAM / RTC Register)

If External RAM is mapped to this memory region, writing to a memory address between $A000 and $BFFF will write to external RAM at `0x2000 * RAM_BANK_NUMBER + (ADDRESS - 0xA000)` if external RAM is enabled. If it is not, the write is ignored.

If an RTC register is mapped to this memory region, writing to a memory address between $A000 and $BFFF will write to the mapped register. For details, check out the [RTC Section](#rtc) of this documentation.

## Memory Reads

### $0000 - $3FFF (ROM Bank 0)

Reading from a memory address between $0000 and $3FFF returns the byte at the corresponding address of the ROM file.

### $4000 - $7FFF (ROM Banks 0x00 - 0x7F)

Reading from a memory address between $4000 and $7FFF returns the byte at `0x4000 * ROM_BANK_NUMBER + (ADDRESS - 0x4000)` of the ROM file.

### $A000 - $BFFF (External RAM / RTC Register)

If External RAM is mapped to this memory region, reading from a memory address between $A000 and $BFFF returns the byte at `0x2000 * RAM_BANK_NUMBER + (ADDRESS - 0xA000)` of external RAM if external RAM is enabled. If it is not, 0xFF is read.

If an RTC register is mapped to this memory region, reading from a memory address between $A000 and $BFFF returns the latched value of the selected register. For details, check out the [RTC Section](#rtc) of this documentation.

## RTC

If the cartridge features a Real Time Clock (indicated by an MBC type identifier of $0F or $10), the RTC-related mechanisms mentioned in the paragraphs above take place.

### RTC Registers

The RTC features a total of 5 registers, each with it's own "Register ID" which is used to determine the register mapped to the $A000 - $BFFF region when writing a value to $4000 - $5FFF. These registers are as follows:

| **Register ID** | **Register Name** | **Register Description**                                     |
| --------------- | ----------------- | ------------------------------------------------------------ |
| $08             | RTC S             | 6-bit register in charge of counting Seconds                 |
| $09             | RTC M             | 6-bit register in charge of counting Minutes                 |
| $0A             | RTC H             | 5-bit register in charge of counting Hours                   |
| $0B             | RTC DL            | 8-bit register representing the lower 8 bits of the total 9-bit Day Counter |
| $0C             | RTC DH            | Bit 0 \| Bit 8 of the Day Counter<br />Bit 6 \| Timer Halt Bit<br />Bit 7 \| Day Counter Carry Bit |

### RTC Clock Ticks

On actual hardware, the RTC on the cartridge is hooked up to a 32.768 kHz Clock. However, this does not necessarily need to be emulated, as the CPU clock works just as well for taking a reference on how much time has passed.

Every 4194304 T-cycles the RTC S register is incremented by 1. If the resulting value is 60, the RTC S register is reset to 0 and the RTC M register is incremented by 1. If the resulting value of this operation is also 60, the RTC M register is also reset to 0 and the RTC H register is incremented. If the resulting value from this is 24, the RTC H register is reset to 0 and the whole 9-bit Day Counter is incremented by 1. If the Day Counter increment causes an overflow, the Day Counter Carry Bit is set to 1. Note that the Carry Bit can only be reset back to 0 by writing 0 to it.

### Latching & Reading RTC Registers

The RTC Data Latch process (described above in the Memory Write $6000 - $7FFF section) "latches" the current state of all RTC registers. This means that it stores the current state in extra registers and makes these registers available to the CPU, while keeping the real RTC registers separate.

If an RTC register is mapped to the $A000 - $BFFF memory region (by writing a Register ID to the $4000 - $5FFF region), reading from this memory region will return the last latched value of the selected register. Unmapped bits (such as the upper 2 bits of the RTC S and RTC M registers) should always read 1. If no latch occurred yet, `0xFF` is read.

### Writing to RTC Registers

By mapping RTC registers to the $A000 - $BFFF memory region, the CPU can also write to the selected RTC register. Values written to RTC registers update both the latched value as well as the value of the actual RTC register. All writes are implicitly bitmasked, as not all bits are actually wired up to the corresponding registers. The following bitmasks are applied:

| **Register Name** | **Applied Bitmask** |
| ----------------- | ------------------- |
| RTC S             | 0b00111111          |
| RTC M             | 0b00111111          |
| RTC H             | 0b00011111          |
| RTC DL            | 0b11111111          |
| RTC DH            | 0b11000001          |

**Note:** Writing to the RTC S register resets the internal sub-second counter. After a write to the RTC S Register, it always takes the RTC exactly one second to increment said register, no matter how much time was left until the next increment before the write.

### The Timer Halt Bit

As soon as the Timer Halt Bit is set to 1 by writing to the RTC DH register, all RTC operation (including sub-second counting) is suspended. This means that, if the RTC is halted 500ms after an RTC S increment, it will take another 500ms for the RTC S register to increment after the RTC is re-enabled.

### Out-of-Range Values

As the 6-bit range of the RTC S and RTC M registers as well as the 5-bit range of the RTC H register exceed their "contextual limit" of 60 and 24 respectively, values above this limit can also be written to these registers. RTC S and RTC M registers may be set to 61 for example, and the RTC H register may be set to 25 or above.

In the case of this happening, the RTC keeps ticking as normal. The values of all registers keep increasing until they reach their limit given by the bit size, at which point they reset back to 0 without incrementing the "following" register. RTC S for example would go from `0x3F` to `0x00` without incrementing the RTC M register. This is due to the fact that the "following-register-increment" only occurs specifically on the values 60 and 24, not on a "greater than or equal" basis.