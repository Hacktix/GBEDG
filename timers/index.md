[<< Back to Last Page](../)

# The Timers

  * [An Introduction](#an-introduction)
  * [Timer Register Overview](#timer-register-overview)
    + [$FF04 - Divider Register (DIV)](#-ff04---divider-register--div-)
    + [$FF05 - Timer Counter (TIMA)](#-ff05---timer-counter--tima-)
    + [$FF06 - Timer Modulo (TMA)](#-ff06---timer-modulo--tma-)
    + [$FF07 - Timer Control (TAC)](#-ff07---timer-control--tac-)
  * [Timer Operation](#timer-operation)
    + [An Example](#an-example)
    + [An Edge Case](#an-edge-case)
    + [TIMA Overflow Behavior](#tima-overflow-behavior)
    + [Why does this work?](#why-does-this-work)

## An Introduction

The purpose of timers within the Gameboy is usually to, as the name would suggest, either measure time or execute segments of code in certain periods of time. In the most general terms, the Gameboy timer consists of four Memory-Mapped I/O registers, two which are linked to the CPU clock and constantly incremented, the other used to configure the behavior of these timer registers. However, a fully accurate implementation of these timers requires much more attention to detail than would initially meet the eye. Everything you need to know is explained in the following few paragraphs.

## Timer Register Overview

### $FF04 - Divider Register (DIV)

To the software, the DIV register appears as an 8-bit register which is incremented every 256 T-cycles. However, the DIV register is the fundamental basis of the entire timer system. Internally, the DIV register is a 16-bit counter which is incremented every single T-cycle, only the upper 8 bits are mapped to memory. The DIV register can be read from at any point in time. However, writing to $FF04 resets the whole internal 16-bit DIV counter to 0 instantly.

### $FF05 - Timer Counter (TIMA)

While the aforementioned DIV register is the "core" of the whole timer system, the TIMA register is the most commonly used software interface. It can be configured using the TAC register at $FF07 to increment at different rates. However, keep in mind that all TIMA increments are bound to the DIV register, as is explained in the [Timer Operation](#timer-operation) section. This register is fully read- and write-capable.

### $FF06 - Timer Modulo (TMA)

The TIMA register can only store 8-bit values. If it overflows, it is reset to the value in this TMA register. Again, there are some oddities to the timing of this operation which is explained in the [TIMA Overflow Behavior](#tima-overflow-behavior) section of this document. Like TIMA, this register is fully readable and writeable.

### $FF07 - Timer Control (TAC)

As the name may suggest, this register controls the behavior of the TIMA register and is also fully readable and writeable. The structure of this register is as follows:

**Bit 2 : Timer Enable** - This bit should be responsible for enabling/disabling increments of the TIMA register, however, it does not quite work as expected all the time. Details can be found further below.

**Bits 1-0 : Clock Select** - These two bits determine at which rate the TIMA register should be incremented, going by the following table:

```
0b00 : CPU Clock / 1024
0b01 : CPU Clock / 16
0b10 : CPU Clock / 64
0b11 : CPU Clock / 256
```

For example: A value of `0b101` would cause the TIMA register to be incremented every 16 T-cycles.

## Timer Operation

This is where things get interesting. As mentioned in the register overview section, the internal 16-bit DIV counter is incremented by 1 every T-cycle. After every increment, the following series of actions occur:

1) A bit position of the 16-bit counter is determined based on the lower 2 bits of the TAC register, as seen here:

```
0b00: Bit 9
0b01: Bit 3
0b10: Bit 5
0b11: Bit 7
```

The bit at the determined position is taken from the 16-bit DIV counter and stored for a later step.

2) The "Timer Enable" bit (Bit 2) is extracted from the value in the TAC register and stored for the next step.

3) The bit taken from the DIV counter is ANDed with the Timer Enable bit. The result of this operation will be referred to as the "AND Result".

The Gameboy searches for a "falling edge" on the AND Result, meaning that it's looking for the value to be `1` in the last cycle, and `0` in the current one. Only if this is the case, the TIMA register is incremented.

### An Example

Let's assume the TAC register is set to `0b101`, which means the Timer is enabled and it is to be incremented at a rate of CPU Clock / 16. The 16-bit DIV counter currently contains the value `0b10111`.

The first cycle is executed and the DIV counter is incremented to `0b11000`. Due to the fact that the "Clock Select" value in TAC is `0b01`, bit 3 of the 16-bit DIV counter (=`1`) is selected for the AND operation. The Timer Enable bit is set to `1`, so the AND instruction ends up being `1 & 1 = 1`. The AND result of the previous cycle was `0`, so this is a "rising edge" (meaning going from 0 to 1). TIMA is not incremented.

Let's continue for a few more cycles. The 16-bit DIV register has been incremented a few times (during which the AND result is always `1`) and now contains `0b11111`. When the next cycle is executed, it is incremented to `0b100000`. Again, bit 3 of the 16-bit DIV counter is selected (=`0`) and ANDed with the Timer Enable bit (=`1`), which results in `0 & 1 = 0`. Since now the AND result is `0`, and the previous AND result was `1`, the TIMA register is incremented.

### An Edge Case

Let's take another example. TAC is still set to `0b101`, so the Timer is enabled and incremented every 16 T-cycles. The 16-bit DIV counter currently contains the value `0b11000`.

A cycle is executed and the counter is incremented to `0b11001`. Bit 3 of the counter (=`1`) is ANDed with the Timer Enable bit (=`1`), resulting in `1 & 1 = 1` - no falling edge, so no TIMA increment.

Now for the next cycle TAC is modified by a memory write and changed to `0b001`, which disables the timer. The DIV counter is once again incremented to `0b11010` and bit 3 (=`1`) is taken for comparison. Now it's the Timer Enable bit which is set to `0`, which results in `1 & 0 = 0`. In the previous cycle the result was one, which means that we have a falling edge despite the fact that the selected bit of the counter didn't change. In this case, TIMA is still incremented.

### TIMA Overflow Behavior

As mentioned before in the Timer Register Overview, when the TIMA register overflows (being incremented when the value is 0xFF), it is "reloaded" with the value of the TMA register at $FF06. Additionally, when this occurs, a Timer Interrupt is requested. Effectively this just means that bit 2 of the Interrupt Flag Register at $FF0F is set.

These actions don't occur instantaneously, however. After overflowing the TIMA register contains a zero value for a duration of 4 T-cycles. Only after these have elapsed it is reloaded with the value of TMA at $FF06, and only then a Timer Interrupt is requested.

The reload of the TMA value as well as the interrupt request can be aborted by writing any value to TIMA during the four T-cycles until it is supposed to be reloaded. The TIMA register contains whatever value was written to it even after the 4 T-cycles have elapsed and no interrupt will be requested.

If TIMA is written to on the same T-cycle on which the reload from TMA occurs the write is ignored and the value in TMA will be loaded into TIMA. However, if TMA is written to on the same T-cycle on which the reload occurs, TMA is updated before its value is loaded into TIMA, meaning the reload will be carried out with the new value.

### Why does this work?

The internal 16-bit DIV counter is incremented with every T-cycle. If we look at the binary numbers of the incrementing register, it should become obvious how this technique works.

```
0b0000
0b0001
0b0010
0b0011
0b0100
0b0101
...
```

Bit 0 alternates every T-cycle, meaning we have a falling edge on every second T-cycle. Bit 1 takes twice as long to alternate - every two T-cycles, giving us a falling edge on every fourth T-cycle. Bit 2 doubles *that* with a falling edge on every 8th T-cycle and Bit 3 once again doubles that with a falling edge on every 16th T-cycle. Now, looking back at the specifications for the Timer Operation, we know that Bit 3 is used for the AND operation when the TIMA rate on TCA is configured to Clock / 16 - the same number of T-cycles it takes to encounter a falling edge. If we spin this structure further to Bit 5, which is checked for an increment rate of Clock / 64, we get a falling edge every 64 T-cycles - and that's what makes the TIMA register tick.

