[<< Back to Last Page](../)

# Basic Emulation Concepts

Before you start to simply "program away", you should try to become familiar with a few basic concepts of emulation. Nothing fancy like design patterns or anything similar, but rather a basic understanding of what the actual hardware does and how you can represent (or "emulate") that behaviour in software. The following paragraphs are intended to give you a short overview and hopefully a little understanding of these concepts.

## What you're emulating

If you've worked on "lower levels" of computer science before this might be obvious to you, but if you haven't it might not. You're not just writing any normal computer program, you're trying to make software that "pretends" to be other hardware. This means that you'll have to deal with quite low-level operations, including bitwise operations such as AND, OR, etc.

Also, things won't always be "straightforward". Real hardware doesn't have as many abstractions as normal software development does, and the more accurately you attempt to emulate a machine, the more complex it's going to get.

## The Concept of Clocks, Cycles & Ticks

In essence, the Gameboy is just a bunch of wires, transistors and other hardware components which, by themselves, don't do a whole lot. What really adds "spice" to the whole hardware mix is the clock - a component that continuously outputs on- and off-signals at a specific frequency. This is what makes the whole system *tick*, as each time the clock starts outputting an on-signal, all the other components do one little thing. It might be as small as adding two bytes together, or drawing one pixel to the screen, or reading a byte from memory. But each of these on-signals making the components do things is called a "tick". A "cycle" is usually just a synonym for a "tick", however, in some aspects, "cycle" and "tick" refer to different things on the Gameboy, but we'll worry about that later.

## How a CPU works (explained in very simple terms)

A major component of any emulator (and usually the first thing you work on) is the main processor, the CPU. While that does sound really fancy, the basic concept (that is usually shared between most, if not all, CPU architectures), is a simple loop called "Fetch - Decode - Execute". Effectively, there's three steps that loop over and over:

* **Step 1:** Fetch
  The processor reads an instruction from memory. Instructions are just one (or more) bytes that tell the CPU what it should be doing.
* **Step 2:** Decode
  The processor takes a good look at the instruction it fetched just before and determines what it should be doing. Once that's done it's time for the last step.
* **Step 3:** Execute
  The processor actually does things now. What exactly it's doing depends on the step just before, maybe it's an addition, maybe it's just reading a byte from memory, or maybe it's just idle and waiting for something to happen.

## The Concept of Memory Addresses

The Gameboy (and pretty much any system) has a whole bunch of memory, which is split up into individual bytes. However, in order to know which byte in memory should be read/written/etc. Memory Addresses are used. If you imagine memory as an array of bytes, a memory address would simply be the index of a byte in that array.

However, it's not always that straightforward. There isn't just one big chunk of memory, there are multiple memory chips of different kinds. This is where "memory mapping" comes into play. Imagine you have two chips with 8 bytes of memory each. The simplest way of making it possible to access both chips with just one address would be to make addresses 0 to 7 access the 8 bytes of the first chip, while addresses 8 to 15 access the 8 bytes of the second chip. The process of splitting up address ranges (aka "Memory Regions") to different memory chips is usually what's meant by "memory mapping".

Memory addresses don't always have to be mapped to bytes in memory chips however. Instead, they can also be mapped to other chips, such as hardware timers, audio chips, etc. This is usually referred to as "Memory Mapped Input/Output" (or MMIO) and is also relevant for emulating the Gameboy.
