# Blinky1541

How to make a program for the Commodore 1541 that blinks the activity LED.


## Introduction

When learning a new programming language, the first program written is typically "Hello, world!".
This assumes a system that can display strings: a `print` library function and a screen to show the printed message.
If the system misses either, the first program is typically "Blinky".
The Blinky program blinks an LED connected to the CPU.

Surely the Commodore 64 falls into the first category (`10 PRINT "HELLO, WORLD!"`).
For sure. However, in this article, we are going to program the _Commodore 1541 disk drive_. 
It has a 6502, RAM, 6522 VIAs and even an LED connected to the VIA. We are going to blink that LED.


## Blinky program

Our first task is to write the Blinky program. We do that in this section.
Next section is about how to get the program on the 1541 and run it there.


### LED control

We want to write a Blinky for the 1541 disk drive.
We need to know _how to control the activity LED_.

On Zimmers we find the 
[schematics of the 1541-ii](https://www.zimmers.net/anonftp/pub/cbm/schematics/drives/new/1541/1541-II.340503.gif). 
Here is an extract.

![LED connection](schematics-actled.png)

- On the 1541-ii the activity LED is _green_ (`grun`) and the power LED is _red_ (`rot`), 
  which is the reverse of the original 1541.
- The green LED is connected with its anode to 5V, so it is _low active_.
- The cathode is connected to port `PB3` of the VIA 6522 (U8).

Also on Zimmers we find the [memory map](https://www.zimmers.net/anonftp/pub/cbm/maps/C1541ram.txt).
Here is an extract

```
VIA 2: 6522, port for motor and read/write head control
------------------------------------------------------

$1C00	PB, control port B
$1C01	PA, port A (data to and from read/write head)
$1C02	CB, data direction port B
$1C03	CA, data direction port A

    PB 7:	SYNC IN
    PB 6,5:	Bit rate D1 and D0
    PB 4:	WPS - Write Protect Sense (1 = On)
    PB 3:	ACT - LED on drive
    PB 2:	MTR - drive motor
    PB 0,1:	step motor for head movement

    CA 1:	Byte ready
    CA 2:	SOE - Set Overflow Enable for 6502
    CA 3:	read/write
```

- At address $1C02 we find the `data direction port B`, it determines wether a pin of port B is input or output.
- At address $1C00 we find the `control port B`, i.e. the value for the pins of port B.
- The table confirms that the `ACT - LED` is PB3.

> Assuming that the initialization of the drive sets the data direction register such that PB3 is output,
> the only thing our Blinky program has to do is _clear_ bit 3 to enable the activity LED or _set_ bit 3 to disable the LED.


### Program location 

In the same [memory map](https://www.zimmers.net/anonftp/pub/cbm/maps/C1541ram.txt) on Zimmers
we find some other interesting addresses:

```
0104 - ff	Stack area

0200 - 29 	Buffer for command string

0300		Buffer 0
0400		Buffer 1
0500		Buffer 2
0600		Buffer 3
0700		Buffer 4

07ff		End of RAM
```

- It tells us that the 1541 has 2 kbyte RAM (8 pages, $0000-$07ff).
- Page 0 (0000-00ff) is in use as zero-page.
- Page 1 (0100-01ff) is used as stack.
- Page 2 (0200-02ff) is for general administration.
- More specificaslly, we see that the command buffer is $29 or 42 bytes.
- Pages 3, 4, 5, 6, and 7 ar sector buffers.

> We will use page 3 (Buffer 0) to store our program.


### The code

Now that we know we need to manipulate bit 3 in 1c00, and 
we decided the store our program in page 3 (0300), we are ready to 
write our Blinky program.

We hand assemble using Maaswerk's [6502 instructions](https://www.masswerk.at/6502/6502_instruction_set.html).

```
0300 | 162,5     | ldx #$5
0302 | 173,0,28  | lda $1c00
0305 | 41,247    | and #$f7
0307 | 141,0,28  | sta $1c00
030a | 32,32,3   | jsr $0320
030d | 173,0,28  | lda $1c00
0310 | 9,8       | ora #$08
0312 | 141,0,28  | sta $1c00
0315 | 32,32,3   | jsr $0320
0318 | 202       | dex
0319 | 208,231   | bne $0302
031b | 96        | rts
031c | 234       | nop
031d | 234       | nop
031e | 234       | nop
031f | 234       | nop
0320 | 169,0     | lda #$00
0322 | 160,0     | ldy #$00
0324 | 136       | dey
0325 | 208,253   | bne $0324
0327 | 56        | sec
0328 | 233,1     | sbc #$01
032a | 208,246   | bne $0322
032c | 96        | rts
```

- At 0320 we have a wait routine. It keeps register X unmodified.
- The wait takes about 0.33 seconds.
- The main routine starts at 0300.
- Register X is used to loop 5 times on/off. it is set st 0300 and decremented at 0318 and looped at 0319.
- At 0302-030a the activity LED is switched on (by clearing bit 3 of 1c00) for 0.33s.
- At 030d-0315 the activity LED is switched off (by setting bit 3 of 1c00) for 0.33s.


## Upload and execute

With our first task completed (writing the Blinky program) this section
focusses on how to get the program on the 1541 and run it there.
We do that via the command channel.


### Commands in general

Recall that we can _send commands_ to a 1541.
We do that by opening a byte pipe (file handle), any will do, here we pick `1`.
The file is on a specific device, here `8` for the disk drive.
TDrives have one channel reserved for commands: `15`.

Here is an example: we send the command `S` to delete ("scratch") a file.
It has as argument `HELLO` (the filename).

```basic
10 PRINT "HELLO, WORLD!"
RUN
HELLO, WORLD!

READY.
SAVE "HELLO",8

SAVING HELLO
READY.
OPEN 1,8,15,"S:HELLO":CLOSE 1

READY.
```

The above command `OPEN 1,8,15,"S:HELLO"` is a shorthand for the following 
which more clearly shows that we send `"S:HELLO"` to file 1, the command channel 15 of drive 8.

```basic
OPEN 1,8,15
PRINT#1,"S:HELLO"
CLOSE 1
```

It is also possible to _receive feedback_ from the drive.
Typically we get back the drive _status_.

```basic
10 OPEN 1,8,15
20 PRINT#1,"S:HELLO"
30 INPUT#1,EN,EM$,ET,ES
40 PRINT EN;EM$;ET;ES
50 CLOSE 1

RUN
 1 FILES SCRATCHED 1  0
```

The above example scratches the file `HELLO` and then checks for success (`1 FILES SCRATCHED`).
It is good to realize that the `INPUT#` function is high level.
It parses the bytes from the disk, using `,` as field separator, 
character 13 (CR) as line separator, and even parses integers 
(e.g. `"00"` becomes `0`). 

For full control it is better to replace `INPUT#` by `GET#` and extract 
the raw bytes one by one.

```basic
10 OPEN 1,8,15
20 PRINT#1,"S:HELLO"
30 GET#1,B$
40 PRINT B$;
50 IF B$<>CHR$(13) THEN GOTO 30
60 CLOSE 1

RUN
01, FILES SCRATCHED,01,00
```

Having said that, `GET#1,B$` has one flaw. If the byte being read is 0, `B$` is `""`
instead of `CHR$(0)`. The usual work around is `30 GET#1,B$:B=0:IF B$<>"" THEN B=ASC(B$)` or the 
slightly faster (without filling the string heap)

```basic
30 GET#1,B$:B=ASC(B$+CHR$(0))
```

### Memory commands


## Links

- Pagetable [Commodore Peripheral Bus: Part 3: Commodore DOS](https://www.pagetable.com/?p=1038).
- Zimmers [memory map](https://www.zimmers.net/anonftp/pub/cbm/maps/C1541ram.txt).
- Zimmers [schematics of the 1541-ii](https://www.zimmers.net/anonftp/pub/cbm/schematics/drives/new/1541/1541-II.340503.gif). 
- Maaswerk [6502 instructions](https://www.masswerk.at/6502/6502_instruction_set.html).

(end)


