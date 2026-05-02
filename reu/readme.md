# REU

In 1985 Commodore released the [REU](https://www.c64-wiki.com/wiki/REU) 
or **R**AM **E**xpansion **U**nit. I got a [Kung Fu Flash 2](https://codeberg.org/KimJorgensen/KungFuFlash2) cartridge 
and that [claims](https://codeberg.org/KimJorgensen/KungFuFlash2#:~:text=Kung%20Fu%20Flash%202%20can%20emulate%20a%201%20Mb%20REU) 
to incorporate a REU. I was wondering if I could use that REU from BASIC.


## Introduction

A REU contains several blocks of RAM, each 64k byte.
The smallest Commodore REU is the "Commodore 1700 REU"; 
it contains 2 blocks of 64 kbyte or 128 kbyte.
The "Commodore 1764 REU" contains 4 blocks or 256 kbyte.
The biggest one from Commodore is the "Commodore 1750 REU", it contains 8 blocks or 512 kbyte.
The "Kung Fu Flash 2" REU contains 16 blocks or 1024 kbyte or 1 Mbyte.
The maximum size possible, fitting to the current REU register API, 
would be 256 blocks of 64 kbyte or 16 Mbyte.

A REU does _not_ use a banking mechanism, in the sense that it 
replaces a part of the C64 RAM by a (selectable) part of the REU.
The REU's memory is _not_ accessible by the 6510 processor of the C64.
Instead the REU is a memory mapped device, 
with 11 control and status registers mapped at 0xDF00.
Via those registers, the 6510 gives the REU a _command_ 
to _stash_ some of the C64's data into the REU,
or to _fetch_ some data from the REU and store it in the C64 RAM.

Technically, I think of the two commands as a `memcpy(dst,src,size)` in hardware, 
where the `src` is in the C64 and the `dst` is in the REU memory (stash)
or vice versa (fetch). The C64 address is two bytes, the REU address is
three bytes and the size is also two bytes.

The copy is performed by the controller in the REU, not by the 6510, the controller of the C64.
It performs the copy at the C64 clock speed - that is what the C64's memory chips can handle.
In other words the REU reads or writes at 1 MHz, or 1 Mbyte per second.
A full C64 memory range (64k) can be read or written in 1/16 second (62.5 ms).

When the 6510 would perform the copy, a typical loop (copying max 256 bytes) 
would be 12 clock cycles: 4 for LDA, 3 for STA, 2 for INX and 3 for BNE.
This means that the 6510 reaches 1 000 000 / 12 or 83 kbyte per second or
65536 bytes in 786 ms. The REU is 12× faster.


## Registers

The REU has the following registers, mapped to address DF00 in the C64's I/O space.

  | Register   | Size | Offset | Hex  | Dec   |
  |:----------:|:----:|:------:|:----:|:-----:|
  | `status`   |   1  |    0   | DF00 | 57088 |
  | `command`  |   1  |    1   | DF01 | 57089 |
  | `c64base`  |   2  |   2,3  | DF02 | 57090 |
  | `reubase`  |   3  |  4,5,6 | DF04 | 57092 |
  | `translen` |   2  |   7,8  | DF07 | 57095 |
  | `irqmask`  |   1  |    9   | DF09 | 57097 |
  | `addrctrl` |   1  |   10   | DF0A | 57098 |

The `c64base` register is 2 bytes; offset 2 is the LSB and offset 3 is the MSB.
The same holds for `translen` (size): offset 7 is the LSB, offset 8 is the MSB.
The `reubase` register is 3 bytes: offset 4 is the LSB, offset 6 is the MSB.
Some documents call the byte at offset 6 a "bank" but that is confusing term.
The REU has a _continuous_ memory from 0x00 0000 to 0xFF FFFF (assuming a 1M byte REU).


## BASIC programs

[reu-presence.prg](reu-presence.prg)

[reu-size.prg](reu-size.prg)


## Link

- [codebase64 REU programming](https://www.codebase64.net/doku.php?id=base:reu_programming)
- [same article on Zimmers](http://www.zimmers.net/anonftp/pub/cbm/documents/chipdata/programming.reu)
- [registers on Zimmers](http://www.zimmers.net/anonftp/pub/cbm/documents/chipdata/reu.registers)

- [c64-wiki REU](https://www.c64-wiki.com/wiki/REU)
- [c64-wiki Commodore REU](https://www.c64-wiki.com/wiki/Commodore_REU)
          
- [Ruud Baltissen E-REU](http://www.baltissen.org/newhtm/e_reu.htm)
- [REU programming by Robin Harbron](https://psw.ca/robin/?page_id=182)


(end)
