[<< Back to Main Page](../)

# The CPU

  * [General](#general)
  * [The Clock](#the-clock)
  * [Registers and Flags](#registers-and-flags)
    + [The Accumulator Register](#the-accumulator-register)
    + [The Flags Register](#the-flags-register)
    + [Initial Register Values](#initial-register-values)
  * [Additional Flags](#additional-flags)
    + [IME (Interrupt Master Enable Flag)](#ime--interrupt-master-enable-flag-)
    + [STATE](#state)

## General
The DMG Gameboy uses an 8-bit Sharp LR35902 CPU, which is a sort of "modified version" of the Z80, intended to fit the lower requirements of a handheld console by using a reduced instruction set.

## The Clock
The main clock of the Gameboy runs at 4.194304MHz. Within the Emulator Development community, one clock cycle of the 4.19MHz clock is often referred to as a "T-Cycle". Many documentations tend to use the term "cycle" ambiguously, as there are also so-called "M-Cycles", which are a duration of 4 T-Cycles. For the sake of consistency, every use of the term "cycle" in this document refers to a T-Cycle.

## Registers and Flags
The CPU has a total of 10 registers (when counting PC and SP), which are used as follows:
```
A          : Accumulator Register (8-bit)
F          : Flags Register       (8-bit)
B, C, D, E : Data Registers       (8-bit)
SP         : Stack Pointer        (16-bit)
PC         : Program Counter      (16-bit)
```

### The Accumulator Register
The A-register effectively acts the same as any data register, however, it is commonly used as the "target register" of various CPU instructions.

### The Flags Register
The F-register contains "status bits" which are set if certain conditions apply after executing an instruction. Which conditions must be met for each bit to be set is up to the instruction itself. The bits are mapped to the 8-bit register as follows:
```
Bit Abbr. Name
 7   ZF   Zero Flag
 6   N    Addition / Subtraction Flag
 5   H    Half Carry Flag
 4   CY   Carry Flag
3-0  -    Unused (always zero)
```

**Note:** The lower 4 bits are *always* zero, no matter what instruction is executed. Example: `POP AF` would pop the value FFFFh onto registers A and F, however, after the instruction has finished, the register pair AF would read FFF0h.

### Initial Register Values
The initial values of each register vary between each system types and model. They are usually defined by the bootrom of the device.

**TODO: Confirm DMG Initial Values**

## Additional Flags
### IME (Interrupt Master Enable Flag)
The IME is a CPU-internal flag which determines whether or not interrupts are enabled. Interrupts are enabled if this flag is set, however, there is a second register mapped to memory which differentiates between various types of interrupts and allows for enabling/disabling each individually.

### STATE
This is mostly relevant for emulator development and is not (?) present in actual hardware. The CPU can be in different states, most notably the default execution state, a HALT state or a STOP state. These will be elaborated on later on in this documentation.
