# RE U

In 1985 Commodore release the [REU](https://www.c64-wiki.com/wiki/REU) 
or RAM expansion unit. I got the [Kung Fu Flash 2](https://codeberg.org/KimJorgensen/KungFuFlash2)
and that [claims](https://codeberg.org/KimJorgensen/KungFuFlash2#:~:text=Kung%20Fu%20Flash%202%20can%20emulate%20a%201%20Mb%20REU) 
to incorporate a REU. I was wondering if a REU is accessible from BASIC.


## Introduction

A REU contains several blocks of RAM, each 64k byte.
The smallest Commodore REU is the "Commodore 1700 REU"; 
it contains 2 blocks of 64 kbytes or 128 kbytes.
The "Commodore 1764 REU" contains 4 blocks or 256 kbytes.
The biggest one is the "Commodore 1750 REU", it contains 8 blocks or 512 kbytes.
The Kung Fu Flash 2 REU contains 16 blocks or 1024 kbytes or 1 Mbyte.
The maximum size possible, fitting to the current REU register API, would be 256 blocks or 16 Mbyte.

A REU does _not_ use a banking mechanism, in the sense that it 
replaces a part of the C64 RAM by a part of the REU.
The REU's memory is _not_ accessible by the 6510 processor of the C64.
Instead the REU is a memory mapped device, 
with 11 control and status registers mapped at 0xDF00.
The C64 gives the REU a command to _stash_ some of the C64's data into the REU RAM,
or to _fetch_ some of the C64's data from the REU RAM.

The REU performs these actions at the C64's clock speed 
(that is what the C64's memories can handle).
This means that the REU reads the C64's bytes (when it performs a stash) 
or writes the bytes (for a fetch) at 1MHz, or 1Mbyte per second.
A full C64 memory range (64k) can be read or written in 1/16 second (62.5ms).


## Registers

The REU has the following registers.

  | Offset | Size |   Hex  | Dec   | Name       |
  |:------:|:----:|:------:|:-----:|:----------:|
  |    0   |   1  | 0xDF00 | 57088 | `status`   |
  |    1   |   1  | 0xDF01 | 57089 | `command`  |
  |   2,3  |   2  | 0xDF02 | 57090 | `c64base`  |
  |  4,5,6 |   3  | 0xDF04 | 57092 | `reubase`  |
  |   7,8  |   2  | 0xDF07 | 57095 | `translen` |
  |    9   |   1  | 0xDF09 | 57097 | `irqmask`  |
  |   10   |   1  | 0xDF0A | 57098 | `addrctrl` |

The `c64base` register is 2 bytes; offset 2 is the LSB, offset 3 is the MSB.
The `reubase` register is 3 bytes; offset 4 is the LSB, offset 6 is the MSB.
The `translen` register is 2 bytes; offset 7 is the LSB, offset 8 is the MSB.

Some documents call the byte a offset 6 a "bank" but that is confusing term.
The REU has a _continuous_ memory from 0x00 0000 to 0xFF FFFF (assuming a 1M byte REU).


## BASIC programs

[reu-presence.prg](reu-presence.prg)

[reu-size.prg](reu-size.prg)




## Link

- [registers](https://www.codebase64.net/doku.php?id=base:reu_programming)
- [same as](http://www.zimmers.net/anonftp/pub/cbm/documents/chipdata/programming.reu)
- [zimmers](http://www.zimmers.net/anonftp/pub/cbm/documents/chipdata/reu.registers)

- [general](https://www.c64-wiki.com/wiki/REU)
- [general](https://www.c64-wiki.com/wiki/Commodore_REU)
          
- [ruud](http://www.baltissen.org/newhtm/e_reu.htm)
- [robin](https://psw.ca/robin/?page_id=182)


(end)
