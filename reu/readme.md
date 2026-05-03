# REU

In 1985 Commodore released the [REU](https://www.c64-wiki.com/wiki/REU) 
or **R**AM **E**xpansion **U**nit. I got a [Kung Fu Flash 2](https://codeberg.org/KimJorgensen/KungFuFlash2) cartridge 
and that [claims](https://codeberg.org/KimJorgensen/KungFuFlash2#:~:text=Kung%20Fu%20Flash%202%20can%20emulate%20a%201%20Mb%20REU) 
to incorporate a REU. I was wondering if I could use that REU from BASIC.

![Kung Fu Flash 2](kungfuflash2.jpg)

If you don't have a kung Fu Flash 2, do not despair. 
The [Commodore 64 Ultimate](https://commodore.net/computer/#:~:text=16%20MB%20system%2C-,16%20MB%20REU,-%2C%2016%20MB%20GeoRAM)
and [TheC64](https://c64os.com/c64os/usersguide/viceconfiguration_thec64#:~:text=Enables%20an%20REU%20with%20the%20maximum%20of%2016MB)
both contains a REU. And even VICE emulates a REU.


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
There are two more command: _swap_ and _compare_, 
but let's postpone those for a while.
This document will use the terms _copy_ and _transfer_ for these
four operations interchangeably. 


## Registers

The [C64 memory map](https://www.c64-wiki.com/wiki/Memory_Map) 
reserves addresses D000-DFFF for I/O.
The REU is typically mapped to region I/O 2 starting at DF00.
The I/O 2 region has 256 bytes, but the REU has only seven registers.
Some registers are 1, some 2 and one is even 3 bytes wide.

  | Register   | Size | Offset | Hex  | Dec   |
  |:----------:|:----:|:------:|:----:|:-----:|
  | `status`   |   1  |    0   | DF00 | 57088 |
  | `command`  |   1  |    1   | DF01 | 57089 |
  | `c64base`  |   2  |   2,3  | DF02 | 57090 |
  | `reubase`  |   3  |  4,5,6 | DF04 | 57092 |
  | `translen` |   2  |   7,8  | DF07 | 57095 |
  | `irqmask`  |   1  |    9   | DF09 | 57097 |
  | `addrctrl` |   1  |   10   | DF0A | 57098 |


### status @0 ($DF00, 57088)

The status register indicates the status of the last transfer.
The status flags (3 MSB) are clear upon read. For smaller transfers, 
the REU is usually so fast (1 kbyte in 1ms, the time of the fastest BASIC statement) 
that checking this bit is superfluous. If the bit is checked, make sure 
to clear it, by reading it, before giving the start transfer command.

The 4 LSB bits are obsolete for modern REUs.

  | Bits | Function          | Details                                                            |
  |:----:|:-----------------:|:-------------------------------------------------------------------|
  |   7  | INTERRUPT PENDING | 1 = interrupt needs servicing; only when INTERRUPT ENABLE is set   |
  |   6  | END OF BLOCK      | 1 = transfer completed; flags completion of all 4 transfer types   |
  |   5  | FAULT             | 1 = compare failed; only for transfer type compare                 |
  |   4  | SIZE              | obsolete (0 for 128k, 1 for >= 256k)                               |
  |  3:0 | VERSION           | obsolete (version number of Commodore REUs)                        |


### command @1 ($DF01, 57089)

  | Bits | Function          | Details                                                            |
  |:----:|:-----------------:|:-------------------------------------------------------------------|
  |   7  | EXECUTE           | writing 1 starts a transfer (of type TRANSFER TYPE)                |
  |   6  | reserved          |                                                                    |
  |   5  | LOAD              | 1 = `c64base`, `reubase`, `translen` are reloaded after completion |
  |   4  | NOFF00            | 1 = start immediately, 0 = wait for write to $FF00; see note       |
  |  3:2 | reserved          |                                                                    |
  |  1:0 | TRANSFER TYPE     | 00=stash (C64 to REU), 01=fetch (REU to C64), 10=swap, 11=compare  |

The transfer type _stash_ transfers data from the C64 memory to the REU memory.
The transfer type _fetch_ transfers data from the REU memory to the C64 memory.
The transfer type _swap_ exchanges the data in the C64 memory with the data in the REU memory.
The transfer type _compare_ compares data from the C64 memory with data in the REU memory
(no writes, only reads). If there is a difference the FAULT-bit in the status register is set. 

> **note**  
> Some memory regions of the C64 are in [triple use](https://www.c64-wiki.com/wiki/Memory_Map).
> For example DF00 could be RAM, I/O 2, or character ROM.
> The REU, when executing a transfer, sees what is configured as active.
> This means that the REU could never access the RAM or character ROM at DF00.
> To solve this, EXECUTE can be postponed by clearing NOFF00.
> The actual transfer is delayed; it starts by writing to FF00, 
> presumably after the memory configuration has changed.


### c64base @2,3 ($DF02, 57090)

The `c64base` is a two-byte register containing the address for the _C64 side_ of the transfer operation.
For a _stash_ command it functions as the _source_ address, for a _fetch_ command it functions as the _destination_ address.
The address is in little endian format: the LSB is at offset 2 and the MSB is at offset 3.


### reubase @4,5,6 ($DF04, 57092)

The `reubase` is a three-byte register containing the address for the _REU side_ of the transfer operation.
For a _stash_ command it functions as the _destination_ address, for a _fetch_ command it functions as the _source_ address.
The address is in little endian format: the LSB is at offset 4 and the MSB is at offset 6.

Some documents call the byte at offset 6 a "bank", with offset 4 and 5 denoting the offset within that 64 kbyte bank.
That is confusing term because the REU has a _continuous_ memory from 0x00 0000 to 0xFF FFFF (assuming a 1M byte REU).


### translen @7,8 ($DF07, 57095)

The `translen` is a two-byte register containing the number of bytes for the transfer operation.
The transfer length is in little endian format: the LSB is at offset 7 and the MSB is at offset 8.


### irqmask @9 ($DF09, 57097)

The REU has two events: transfer completed and compare failed.
When such an event happens, the associated bit in the status register 
is set (END of BLOCK respectively FAULT).

It is possible to generate an interrupt (IRQ, not NMI) when these events occur.
To enable these interrupts, make sure their enable mask is set in `irqmask`.
Secondly, globally enable interrupts by setting INTERRUPT ENABLE.

  | Bits | Function          | Details                                                            |
  |:----:|:-----------------:|:-------------------------------------------------------------------|
  |   7  | INTERRUPT ENABLE  | 1 = trigger IRQ when one of below events happen and is enabled     |
  |   6  | END OF BLOCK MASK | 1 = enable IRQ when status END OF BLOCK is set                     |
  |   5  | FAULT MASK        | 1 = enable IRQ when status FAULT is set                            |
  |  4:0 | reserved          |                                                                    |

If an event happens, and its mask is set, and INTERRUPT ENABLE is set, the IRQ is fired.
It is vectored in hardware at FFFE. The C64 kernal ROM maps that to FF48, which pushes 
the registers and then vectors via $0314/$0315 (for a hardware IRQ; alternatively via 
$0316/$0317 for a BRK, as determined via the B flag in PSW). By default 314/315 
routes to $EA31 (e.g. keyboard scan). One would need to write an ISR (REU handler) 
which **clears the interrupt** by reading $DF00, and then continuous to $EA31. 
If the interrupt is not cleared, it will fire again as soon as the RTI at the end of 
the EA31 ISR is executed.

Although this interrupt mechanism exists, it is fairly useless.
The 6510 is halted during the REU transfer and once the REU transfer is completed, 
the 6510 continuous. One could add a `WAIT 57088` after the `POKE 57089`.


### addrctrl @10 ($DF0A, 57098)

The registers `c64base`, `reubase`, and `translen` are configured before an EXECUTE.
During the transfer the base registers are stepped `translen` times to advance through 
the memories. However, it is also possible to not step the addresses, but keep them fixed.
This is useful in case of a memory fill (memory clear). Another use case is when the
C64 address is fixed and this is a hardware register for sound or a GPIO pin.
Then the REU is a DMA engine that drives those hardware peripherals.

  | Bits | Function          | Details                                                            |
  |:----:|:-----------------:|:-------------------------------------------------------------------|
  |   7  | C64BASEFIX        | 1 = fixed (e.g. for memory fill), 0 = stepping (copy, compare)     |
  |   6  | REUBASEFIX        | 1 = fixed (e.g. for memory fill), 0 = stepping (copy, compare)     |
  |  5:0 | reserved          |                                                                    |


## Timing

The transfer is performed by the controller in the REU, not by the 6510, the controller of the C64.
It performs the copy at the C64 clock speed - that is what the C64's memory chips can handle.
In other words the REU reads or writes at 1 MHz, or 1 Mbyte per second.
A full C64 memory range (64k) can be read or written in 1/16 second (62.5 ms).

When the 6510 would perform the transfer, a typical loop (copying max 256 bytes) 
would be 12 clock cycles: 4 for LDA, 3 for STA, 2 for INX and 3 for BNE.
This means that the 6510 reaches 1 000 000 / 12 or 83 kbyte per second or
65536 bytes in 786 ms. The REU is 12× faster.

A swap transfer is twice as slow since it needs a read and a write of every location.


## Tests

In this section, we are going to try-out the registers of the REU.


### Introduction`

To follow in VICE, goto Preferences > Settings > Cartridges > RAM Expansion Module.
Select a Size. Also note that the REU memory can be saved to a file.
That is handy for our experiments.

![VICE REU](vice-reu.png)

On Kung Fu Flash we have to be sure to end up in a mode where the REU is enabled.
See [KimJorgensen repo](https://codeberg.org/KimJorgensen/KungFuFlash2#reu-emulation).


### Presence

I struggled with enabling my REU on the Kung Fu Flash 2.
I wrote several tests to test if the REU is present (active).
I have combined all tests in one program: 01-presence.

```
100 rem reu presence tester
110 rem mc pennings, 2026 may 3
120 print"{clr}{down}{wht}testing presence of reu{lblu}"
130 r=57088:f=0:rem reu addrs,num fails
140 :
150 :
200 print "{down}test 1 reu datalines float"
210 v=peek(r):print " status: peek";v;
220 if v=255 then print "floats:maybe no reu":f=f+1:goto 300
230 print "not floating:maybe reu"
290 :
295 :
300 print "{down}test 2 set endofblock-mask"
310 poke r+9,64:v=peek(r+9)and224
320 print " mask: poke 64 peek";v;
330 if v<>64 then print "unequal:no reu":f=f+1:goto 400
340 print "equal:maybe reu"
390 :
395 :
400 print "{down}test 3 set fault-mask"
410 poke r+9,32:v=peek(r+9)and224
420 print " mask: poke 32 peek";v;
430 if v<>32 then print "unequal:no reu":f=f+1:goto 500
440 print "equal:maybe reu"
490 :
495 :
500 print "{down}test 4 clr endofblock-mask"
510 v=peek(r)and224:rem read clears
520 print " status: peek";v;"(clr) ";
530 if v<>0 then print "unequal:no reu":f=f+1:goto 600
540 print "equal:maybe reu"
590 :
595 :
600 print "{down}test 5 transfer"
610 poke r+2,0:poke r+3,192:rem $c000
620 poke r+4,0:poke r+5,0:poke r+6,0
630 poke r+7,5:poke r+8,0:rem len=$0005
640 poke r+9,0:rem int mask
650 poke r+10,0:rem inc both addrs
660 poke r+1,128+16:rem ex+noff00+stash
670 v=peek(r):print " status: peek";v;
680 if (v and 64)<>64 then print "eob not set:no reu":f=f+1:goto 700
690 print "eob set:maybe reu"
695 :
697 :
700 print "{down}test report (";f;"fails )"
710 if f=0 then print" all tests pass"
720 if f<>0then print" some tests fail"
730 if f=0 then print" reu {wht}present{lblu}"
740 if f<>0then print" reu {wht}absent{lblu}"
```

- Test 1 (line 200) reads the memory location where the REU is expected (`R` or 57088).
  If there is no device, typically the data line are floating and the 6510 reads 255.
  
- Test 2 (line 300) goes a step further and performs a write and read back of the REU register `irqmask`.
  The test writes 64 and reads that back. Of course there might be a non-REU device 
  that happens to read 64 at address `R+9`. Hence test 3.
  
- Test 3 (line 400) is the same as test 2, but now writes 32.
  This establishes that `R+9` is a register that can be written and read back.
  Could still be another device than REU.
  
- Test 4 (line 500) is a preparation step for test 5. It reads the REU and assumes the 
  END-OF-BLOCK is clear - since no transfer has taken place.
  A read will also clear it, for test 5.
  
- Test 5 (line 600) is a full transfer command from C000 to 000000 of 5 bytes.
  This should set the END-OF-BLOCK in `status`.
  The change that another device then a REU has this action-result behavior is small.

- Line 700 presents the conclusion.
  
![01-presence](01-presence.png)


### Size

The `status` register has one bit indicating size.
If `status.SIZE` is 0 the REU is 128 kbyte, otherwise it is larger.

I struggled, again, to determine REU size.
The problem is that a write to an address beyond the size of the REU is successful; 
the correct value can even be read back. The reason for this is that the REU 
"wraps around" for high addresses. For example, if the REU contains 4 blocks of 
64 kbyte, a write to block 4 ends up in block 0, a write to 5 ends up in 1, 6 in 2,
and 7 in 3. Even more, a write to block 8 also wraps to block 0.

I solved the problem of finding a size is to do a single byte write 
to the highest address of every block. The value I write is the block number.
In other words, I write 00 to 00FFFF, 01 to 01FFFF, 02 to 02FFFF and so, up to FF to FFFFFF.
This takes 256 transfers which can be done in reasonable time. 
The second part of the solution is to write these values in reverse order:
starting with block FF, then FE, down to 00. The effect of this is that
(the highest byte of a) block contains the block number.
The higher blocks also write their high block number, but these get overridden 
by the writes of the lower blocks.

Below table shows the content of (the highest byte of) the REU blocks.
It assumes a REU with 4 blocks (columns 0, 1, 2, and 3), but also shows the 
virtual blocks (4, 5, 6, 7 which could have been extended to 255).
All virtual blocks are enclosed in parenthesis.
The first row is at step 248, when blocks 255 down to 8 have been written.
The next rows show the final steps, until the last block, block 0, is written.

  | time (action) \ block |  0  |  1  |  2  |  3  | (4) | (5) | (6) | (7) | ... | 
  |:----------------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
  | time=248: stash 8     |  8  |  9  | 10  | 11  | (8) | (9) |(10) |(11) | ... |
  | time=249: stash 7     |  8  |  9  | 10  |  7  | (8) | (9) |(10) | (7) | ... |
  | time=250: stash 6     |  8  |  9  |  6  |  7  | (8) | (9) | (6) | (7) | ... |
  | time=251: stash 5     |  8  |  5  |  6  |  7  | (8) | (5) | (6) | (7) | ... |
  | time=252: stash 4     |  4  |  5  |  6  |  7  | (4) | (5) | (6) | (7) | ... |
  | time=253: stash 3     |  4  |  5  |  6  |  3  | (4) | (5) | (6) | (3) | ... |
  | time=254: stash 2     |  4  |  5  |  2  |  3  | (4) | (5) | (2) | (3) | ... |
  | time=255: stash 1     |  4  |  1  |  2  |  3  | (4) | (1) | (2) | (3) | ... |
  | time=256: stash 0     |  0  |  1  |  2  |  3  | (0) | (1) | (2) | (3) | ... |

At the end, blocks 0, 1, 2 and 3 have the correct value, but block 4 is wrong.
It should have 4 but has 0. So the REU size is 4.

- Note that `CS` has the LOAD flag set (line 140). 
  This means the `c64base`, `reubase` and `translen` 
  are restored after each transfer, which allows us 
  to only poke `R+6` in each loop.
- Both loops (of 256 iterations) print a progress bar with `:` 
  every 8 (see `P`) iterations.
- The second loop abort as soon as the REU memory ends (line 440).
- On line 430 we "clear" the C64base location with `255-B`;
  we expect to read `B`.

```
100 rem reu size tester
110 rem mc pennings 2026 may 3
120 print "reu size tester"
130 r=57088:a=49152:rem r=reu,a=c64base
140 cs=128+32+16:rem ex+ld+noff00+stash
150 cf=cs+1:rem fetch
160 :
200 rem prep reu
210 poke r+2,0:poke r+3,192:rem c000=a
220 poke r+4,255:poker+5,255:rem r+6
230 poke r+7,1:poke r+8,0:rem len=0001
240 poke r+9,0:rem int mask
250 poke r+10,0:rem inc both addrs
260 :
300 rem writing 256 blocks
310 print "stash :";:p=8
320 for b=255 to 0 step -1
330 :pokea,b:poker+6,b:poker+1,cs
340 :p=p-1:if p=0 then p=8:print":";
350 next:print
360 :
400 rem reading and checking blocks
410 print "fetch :";:p=8
420 for b=0 to 255
430 :pokea,255-b:poker+6,b:poker+1,cf
440 :if peek(a)<>b then print:goto 500
450 :p=p-1:if p=0 then p=8:print":";
460 next:print
470 :
500 rem report size
510 b1$=mid$(str$(b),2)
520 b2$=mid$(str$(b*64),2)
530 b3$=mid$(str$(b/16),2)
540 print "reu "b1$"*64k "b2$"k "b3$"m"
```

![02-size](02-size.png)


### Todo

- show clear of status by read
- show end of block
- show fault
- show interrupt pending (poke $0314/5 for ISR location. in ISR LDA DF00, JMP EA31)
- WAIT 57088, 64 for completion

- test stash
- test fetch
- test compare
- test swap
- test LOAD
- test FF00
- continuous REU memory


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

- [BASIC control characters](https://www.c64-wiki.com/wiki/control_character)
(end)
