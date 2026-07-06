# C64howto

Various "How to xxx" questions and answers for the C64 (Commodore 64) that I want to remember. Some are linking to my old [howto repo](https://github.com/maarten-pennings/howto).


## Blinky on a 1541

How to make a program for the Commodore 1541 that blinks the activity LED.
See [blinky1541](blinky1541).


## Modern 1541 disk drive

The 1541 replacement that best mimics an 1541, is _Pi1541_ made by Steve White.
Pi1541 runs the original Commodore 1541 firmware on a Raspberry Pi emulating an 6502 and all other chips inside the 1541.
Steve separated the [documentation](https://cbm-pi1541.firebaseapp.com/) 
from the [firmware](https://github.com/pi1541/Pi1541).
I made one, see the associated [repo](https://github.com/maarten-pennings/Pi1541device).


## Modern joystick

With a bit of bridging electronics, 
you can turn a Nintendo Nunchuk into a C64 joystick.
See my [repo](https://github.com/maarten-pennings/Nunchuk) dedicated to the Nunchuk for C64.


## Modern datasette

There is a modern take on the old commodore datasette. 
It is called _Tapuino_, made by "sweetlilmre" (Peter Edwards), 
see [repo](https://github.com/sweetlilmre/tapuino).
Tapuino is basically a digital tape recorder, using an 
Arduino Nano as AD/DA and SD card controller.
I made one, see the associated [repo](https://github.com/maarten-pennings/Tapuino).


## REU (RAM Expansion Unit)

The C64 can be equipped with a REU to expand its memory.
I was wondering [how to](reu) use the REU (from BASIC).


## 9V AC at 50 or 60 Hz

I got a retrofit power supply for my C64 with a switch for 50 or 60 Hz.
I needed to [study](9VAC50vs60) this to understand what it means.


## C64 development

It is possible to develop on the C64 itself. For BASIC that is possibly the best solution.
For assembly it starts to be less than ideal, if only because a bad program may crash your C64, 
corrupting your work.

The [C64 development (old repo)](https://github.com/maarten-pennings/howto/tree/main/c64development) 
explains how to develop assembler and BASIC on a PC, and test it in VICE.


## C64 disk drive commands

[How to (old repo)](https://github.com/maarten-pennings/howto/tree/main//c64diskcmds) manage in C64 disk drive.
The DOS (Disk Operating System) built into the drive.


## How to fix the RESTORE key on the C64

On the older C64 you really have to quickly hammer that key.
Here is an instruction [how to (old repo)](https://github.com/maarten-pennings/howto/tree/main//c64RESTOREkey/readme.md) fix that.


## How to map keys for BMC64

I made a bare-metal-C64 using a Raspbery Pi and a small USB keyboard.
I wanted a mapping that is close to the original C64 layout.
The [BMC64 KBD (old repo)](https://github.com/maarten-pennings/howto/tree/main/c64-bmc64kbd) explains how I did that.


## How to use the SID of the Commodore 64?

Writing a UI to control the SID registers, and using the VICE tools
to extract the BASIC program to PC, see [c64sid (old repo)](https://github.com/maarten-pennings/howto/tree/main/c64sid).


## How to use the compare instruction of the 6502?

I was writing some assembly on the 6502 in the C64 (ok the 6510)
and tripped over the compare function.
Why doesn't that [work (old repo)](https://github.com/maarten-pennings/howto/tree/main/c64cmp)?


## Commodore 64 user defined functions

A rarely used feature of the C64 is the user defined function `FNF(X)`.
Article [c64functions (old repo)](https://github.com/maarten-pennings/howto/tree/main/c64functions) studies how to use them.


## Commodore 64 memory partitioning

Can we partition the C64 memory, storing two programs simultaneously?A multi-app setup?
No multitasking, we activate one, run it, activate another and run it.
The howto [c64mempart (old repo)](https://github.com/maarten-pennings/howto/tree/main/c64mempart) explains it.


## Commodore 64 characters

How are [C64 chars (old repo)](https://github.com/maarten-pennings/howto/tree/main/c64chars) organized?


## Commodore 64 USR() function

How to make the `USR()` function work. Plus, as bonus, a return-to-TMP (Turbo Macro Pro) easily.
See [c64usr (old repo)](https://github.com/maarten-pennings/howto/tree/main/c64usr).


## Commodore 64 string storage

How does the C64 store strings. Section [C64strings (old repo)](https://github.com/maarten-pennings/howto/tree/main/C64strings) explains this.


## Commodore 64 drives

The 1541 disk drive for the commodore 64 is "smart".
This [how to (old repo)](https://github.com/maarten-pennings/howto/tree/main/c64drive) explains what that means.


## Remapping keys in VICE

I was playing with VICE the C64 (Commodore 64) emulator.
In Turbo Macro Pro, the `←` key is used extensively, so I wanted to remap that to an easy key.
See [how to (old repo)](https://github.com/maarten-pennings/howto/tree/main/ViceKeyboardRemap/readme.md) do that yourself.


(end)
