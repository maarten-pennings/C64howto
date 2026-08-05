# Commodore 64 rows versus lines

Lines in a Commodore 64 BASIC program can be up to 80 characters. 
Not more. A line longer than 40 characters is spread over two rows of the screen.
That two rows can form one line is not only a feature of BASIC, 
it is also a feature of the programming environment; or should I say of 
the C64 "terminal".

In this article we have a look at how C64 treats rows and lines.


## Hello program

We enter the following program, but with one special step.
Lines 100 and 110 are terminated with an ENTER, 
but line 200 is extended with 6 spaces and then we 
continue entering line 210 which we terminate with an ENTER.

> This program is listed in lower case to make copy&paste to VICE easier.
> It is available as `HELLO` on the [disk](rowsvslines.d64).

```
100 print "hello, world!":rem long
110 print "hi":rem short
200 print "hello, again!":rem long
210 print "hey":rem short
```

![Entering the HELLO program](hello1.png)

When we `LIST` the program it looks "normal"; the same as how we entered it.

![Listing the HELLO program](hello2.png)

But when we run it, we miss the `HEY` from line 210.

![Running the HELLO program](hello3.png)

When we list the individual lines, line 100 and 110 are fine.
But `LIST 200` also lists `210`, and `LIST 210` lists nothing.

![Lines of the HELLO program](hello4.png)

What happens here is that line 210 is actually part of the `REM` comment of 
line 200, but the careful placement of 6 spaces hides that. When we edit line 
200 and delete 5 spaces, it is much more clear how the program really looks like.

![Editing line 200 the HELLO program](hello5.png)

You probably knew this.
And you probably found out there is no way to "split" line 200 in two lines.
In a modern editor an ENTER in the middle of a line splits at cursor.
But for the C64 terminal, ENTER is "commit" (the line or the command).

The only way to split is to `LIST` the line to be split, delete the part you don't want, 
prepend a fresh line number and pres ENTER. And then `LIST` the original line again and 
delete the part you just copied to the fresh line and pres ENTER again.


## Scrolling 

What I did notice in the past, but never fully realized, is that scrolling 
is impacted by lines spanning two rows.

The following program has two lines each fitting on one screen row (100, 120) 
and one line that spans two rows (110).

> This program is listed in lower case to make copy&paste to VICE easier.
> It is available as `SCROLL` on the [disk](rowsvslines.d64).

```
100 print "hello, world!"
110 print "this is a line that spans two rows"
120 print "short again"
```

The double row span is hard to see from the listing above, but easy to 
spot on the C64 screen (first screen shot). 

The series of screenshots starts with the cursor at the bottom of the screen.
Each screenshot we press ENTER once to go to the next screenshot.

![Repeatedly pressing RETURN for the SCROLL program](scroll1-6.png)

Observe that each ENTER at the bottom scrolls _one_ row up, 
as long as the first row of the screen is a complete (one-row) line.

In screenshot 3, the top of the screen has a line that spans two 
screen rows, an ENTER now scrolls _two_ rows. Also notice that 
not only the top two rows scroll off the screen, but there is 
also an irregularity at the bottom: the cursor moved one row up!

In other words: **scrolling is not per row but per line** (at the top of the screen).


## 10 PRINT

Recall the famous 10PRINT program.

```
10 PRINT CHR$(205.5+RND(1));:GOTO 10
```

It  suffers from "scrolling is not per row but per line".
To show that, we have written an extended version that pauses 
each time the cursor reaches the bottom right corner of the screen.

> This program is listed in lower case to make copy&paste to VICE easier.
> It is available as `10PRINT` on the [disk](rowsvslines.d64).

```
100 i=rnd(-123)
110 for i=1 to 24*40-1
120 :print chr$(205.5+rnd(1));
130 next i
140 :
200 for t=0 to 9999:next t
210 :
300 for i=1 to 40
310 :print chr$(205.5+rnd(1));
320 next i
330 :
400 goto 200
```

We have made a series of screenshots - one at each scroll.

![Screenshot of scrolling the 10 PRINT program](10print1.png)

Note that ever other scroll is _two_ rows instead of one 
(see the solid blue row at the bottom of the screen in 
shot 3, 5, and 7).

You might wonder why.
The 10PRINT program prints character after character never 
ending the line: every `PRINT` ends with a semicolon (`;`).
So after 40 PRINTs the first screen _row_ is full, but the _line_
continues. After 80 PRINTs also the second screen _row_ is full,
and still the _line_ continues. 

However the C64 terminal can not handle lines longer than two rows.
So the line is aborted, and a new line is started on the third row.
That one continues till the fourth row, and again a new line is 
forced by the terminal. As a result the screen is filled with 
several lines of 80 characters, each occupying 2 rows.

As a result the scrolling of 10PRINT is two rows, one row, two rows, one row.

The "scrolling is not per row but per line" might sound reasonable.
The screen never starts with the second half of a line.
However, I think I could live with half lines at the top and have smoother scrolling. 
I guess this is just an implementation choice; it probably made 
sense (easier terminal code?), and we have to live with it.


## Line link table

How does the terminal know that two rows form one line?
Enter the _line link table_, stored at addresses $00D9-$00F2.

For details of the line link table, see e.g. the famous book
[Mapping the 64](https://www.pagetable.com/c64ref/c64mem/#:~:text=Screen-,Line%20Link,-Table/Editor%20Temporary).
We will simplify it here.

The line link table has one byte per row.
Bit 7 at offset _r_ in the line link table is the link flag for row _r_.
That flag is _set_ when that row contains the first half of a line (or an entire line).
That flag is _clear_ when that row contains the second half of a line.

The first row has its link flag at address $00D9 or 217, 
the second row at $00DA or 218, etc.

The following program demonstrates the line link table.
The first half of the program prints a screen full of lines (program lines 100-190).
Some output lines will be less than 40 characters (one row), some will be less 
than 80 (two rows) and some are even longer (3 rows) - maximum is 40×3½ characters. 

We use the characters from the 10PRINT program to fill the lines.
Line 130 tags the _beginning of each line_ with an increasing 
letter (`A`, `B`, etc).

Line 110 seeds the random number generator so that each run produces 
the same output; delete it if you don't want that.

> This program is listed in lower case to make copy&paste to VICE easier.
> It is available as `LINELINKTABLE` on the [disk](rowsvslines.d64).

```
100 rem print lines with random length
110 c=rnd(-123)
120 for l=0 to 12
130 :print chr$(65+l);
140 :for c=0 to rnd(1)*40*3.5
150 ::print chr$(205.5+rnd(1));
160 :next c
170 :print
180 next l
190 :
200 for l=0 to 24
220 :lk=(peek(217+l) and 128)=0
230 :poke 55296+l*40,lk+1
240 next l
250 get a$:if a$="" then 250
```

The interesting part is the second half of the program (program lines 200-250)
which runs once the screen is filled. It loops over all rows (L from 0 to 24)
and inspects the line link table entry for that row: `PEEK(217+L)`.
Variable `LK` is true if the link flag is clear, e.g. when that row contains 
a second half. Recall that true is -1 in C64 BASIC and false is 0 so `LK+1` 
is 0 (corresponding to color "black") for a second half and 1 (i.e. "white") 
for a first half. This value is POKEd on line 230 as the character color for 
the first character of each row. Line 250 waits for a key press, ending the 
program, leading to `READY` and a scroll.

![The output of LINELINKTABLE](linelinktable1.png)

The output is as expected.
Line `A` is short, two characters, it fits on one row, so `A` is white.
Line `B` is extra long, more than 2 rows. The first row is the first part of the line 
so `B` is white. The next row is the second part of the `B` line so the slash is black. 
The next row is the third part of the `B` line. That third part did not fit on the 
first two rows, so the terminal started a new row, hence the slash is white.


## Better 10PRINT

It is also possible to _write_ to the line link table.
We can break long lines (two rows) into two parts (two rows).
One advantage is smoother scrolling.

Find below an improved version of the 10PRINT program.
Every 40 characters printed, it splits row 1 from row 0 by 
setting the link flag of row 1. This results in smooth scrolling.

> This program is listed in lower case to make copy&paste to VICE easier.
> It is available as `10PRINTSMOOTH` on the [disk](rowsvslines.d64).

```
0 fori=1to40:printchr$(205.5+rnd(1));:next:poke218,128:goto
```

Note that if `GOTO` has no line number it jumps to line 0, a small optimization.

When you run this program it scrolls one row at a time.
Remove the POKE and it alternates between scrolling one and two rows.

The line link table contains other bits. A safer variant would be
`POKE 218,PEEK(218) OR 128` but this is slower, and the row is 
scrolled out soon anyhow...

## Links

- [D64image](rowsvslines.d64) of all BASIC programs in this article.
- Line link table in [Mapping the 64](https://www.pagetable.com/c64ref/c64mem/#:~:text=Screen-,Line%20Link,-Table/Editor%20Temporary).
- 10PRINT [website](https://10print.org/).

(end)
