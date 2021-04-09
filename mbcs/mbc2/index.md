[<< Back to Last Page](../)

# MBC2

The MBC2 is quite a rare MBC which supports up to 16 ROM banks (256KB). It has no support for variable-size external RAM, and instead always comes packaged with 512x4 bits of RAM.

## MBC Capabilities

The MBC only needs to keep track of one value:

* **ROM Bank Number:** Unsigned 4-bit number which determines which 16KB block of ROM is mapped to the memory region $4000 - $7FFF.

If the MBC has a battery (MBC type identifier $06), the state of external RAM should be stored to a file when the emulator closes and reloaded once it starts the same game again.

## Memory Writes

### $0000 - $3FFF (Enable RAM / ROM Bank Number)

Whether a write to this region enables/disables RAM or changes the ROM Bank Number depends on Bit 8 of the memory address that is written to.

If Bit 8 of the memory address is 0, the write enables RAM if the lower 4 bits of the written value are equal to `0xA`. If they are not, external RAM will be disabled.

If Bit 8 of the memory address is 1, the write changes the ROM Bank Number to the written value. If the value is zero, the ROM Bank Number will be set to 1 instead.

### $A000 - $BFFF (External RAM)

Writing to a memory address between $A000 and $BFFF will write to external RAM if it is enabled. If it is not, the write is ignored.

As the MBC2 always has 512 accessible addresses, only the lowest 9 address bits are actually wired up to RAM. This means that addresses $A000 and $A200 access the exact same value in RAM. The actual address can be calculated by ANDing the address with `0x1FF`.

Since each memory address can only hold 4 bits, only the lowest 4 bits of the written value are actually stored in RAM. All other bits are ignored.

## Memory Reads

### $0000 - $3FFF (ROM Bank 0)

Reading from a memory address between $0000 and $3FFF returns the byte at the corresponding address of the ROM file.

### $4000 - $7FFF (ROM Banks 0x1 - 0xF)

Reading from a memory address between $4000 and $7FFF returns the byte at `0x4000 * ROM_BANK_NUMBER + (ADDRESS - 0x4000)` of the ROM file.

### $A000 - $BFFF (External RAM)

Reading from a memory address between $A000 and $BFFF will return the 4 bits at `ADDRESS & 0x1FF` of external RAM if external RAM is enabled. If it is not, 0xFF is read. The upper 4 bits of all read values are constantly 1, as they are not actually wired up to the RAM chip.