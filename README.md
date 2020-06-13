# Welcome to the GameBoy Emulator Development Guide!
This documentation is intended to assist with development of emulators for the original DMG GameBoy.

**Note:** Everything in this documentation is heavily Work-In-Progress and may have to be taken with a grain of salt.

## The Development Journey

1) Before beginning development of your emulator, you may want to check out a general overview of the [Hardware](hardware) you'll be emulating. Getting everything set up in the right structure to account for every piece of hardware first is going to make everything easier in the long run.

2) Once you feel ready to start, the best place to do so would be emulating the [CPU](cpu). This will make sure the core logic in both your test ROMs as well as games is executed correctly and, well, *at all*.

3) With the CPU implementation in place, it's time to implement a first [Memory Bank Controller](mbcs). As most of the well-known test ROMs use [MBC1](mbcs#MBC1), it would be best to implement that one first.

4) Now it's time to test what you've made. It's practically impossible to implement everything correctly on the first try, so a good bit of debugging should be expected. Also, following these [CPU Testing Guidelines](cputest) may help in finding issues easier and faster.

**TODO: Extend list**

## Quick Reference

[Hardware Overview](hardware)

[CPU Structure](cpu)

[Memory Bank Controllers](mbcs)

[CPU Testing Guidelines](cputest)