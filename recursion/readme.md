# Recursion in BASIC

Can we use _recursion_ in Commodore 64 BASIC?

As this article explains, the answer is yes.
However, recursion is a bit harder than in more modern 
programming languages,


## Introduction

Commodore 64 BASIC was my first programming language.
I didn't know the terminology yet, but I learned the _assignment statement_ 
(`LET`), that it knows about the order of the operations (`*` before `+`) and 
that it uses and changes the value of _variables_. I learned that you can `INPUT` 
and `PRINT` variables. 

There was the relatively high-level `FOR`-`NEXT` statement. 
Then there was the `IF` statement. It is less high-level because 
the "room" for the `THEN` is one line and there is no `ELSE` branch.
Both shortcomings can only be remedied if you're willing to throw 
in a low-level `GOTO`. 

And finally there was the magic `GOSUB`, the `GOTO` that remembers 
to `RETURN` where it came from. Made for _subroutines_.

I had no idea you could have named subroutines ("functions") nor that 
subroutines could have arguments or use local variables (on a stack).
As a result I had no clue that a thing called _recursion_ existed.

I learned about recursion much later. Looking back, I wondered if recursion 
is at all possible on the C64. Not having subroutine arguments on a 
stack seems blocking.

In this article we first look at three toy problems that we solve 
recursively in C64 BASIC. They learn us the concepts we need for 
real problems. Then we have a quick detour looking at the C64 stack.
Finally we solve three bigger recursively.


## Theory

In this chapter we use toy problems to understand how to apply recursion 
in BASIC. None of the presented problems should be solved recursively.
The stack soon becomes a limiting factor, and execution speed is also 
heavily impacted.

But the presented problems are well known, and easy, so they don't 
distract us when we learn the recursion concepts.


### Power of 2

The first problem we are going to solve recursively is to compute 
powers of two. More specifically, we want a _subroutine_ that computes 
the value `2^N` recursively.

In BASIC we don't have functions with arguments and a return value, so 
we need to use global variables. Our subroutine will have an input `N`.
This means that the caller must assign `N` before the `GOSUB` to 
the subroutine. The subroutine will (recursively) compute `2^N` and 
assign that to the global variable `R` (from "result").

The above paragraph describes the "signature" of the subroutine:
which values go in (in which variables) and which values go out 
(in which variables). One thing is still missing in the "signature" 
description: which global variables are used as scratch pad, i.e. 
modified (overwritten) by the subroutine, but not having a meaningful 
value after return. 

In our case, `N` is modified. The signature of the subroutine is 
specified on line 200.

> This program is listed in lower case to make copy&paste to VICE easier.
> It is available as `1-POW2` on the [disk](recursion.d64).

```basic
100 print "pow2"
110 i=0
120 :n=i:gosub 200:print "2^";i;"=";r
130 :i=i+1
140 goto 120
150 :
200 rem inp:n; out:r=2^n; mods:n
210 if n=0 then r=1:return
220 n=n-1
230 gosub 200
240 r=r*2
250 return
```

The body of the subroutine (lines 210-250) 
implements the base case (`N=0`, line 210) and the recursive case 
(`N>0`, lines 220-250):

```
pow2(0)= 1
pow2(n)= pow2(n-1)*2, if n>0
```

That was not too hard.

Lines 100-140 test the subroutine.
Since `N` is modified by the subroutine, we can not use `N` in the 
main loop, hence we use a fresh loop variable `I`.
On line 120, the variable (argument) `N` is defined before the `GOSUB`, 
afterwards the variable (result) `R` is used (to `PRINT`).

```
pow2
2^ 0 = 1
2^ 1 = 2
2^ 2 = 4
2^ 3 = 8
2^ 4 = 16
2^ 5 = 32
2^ 6 = 64
2^ 7 = 128
2^ 8 = 256
2^ 9 = 512
2^ 10 = 1024
2^ 11 = 2048
2^ 12 = 4096
2^ 13 = 8192
2^ 13 = 8192
2^ 14 = 16384
2^ 15 = 32768
2^ 16 = 65536
2^ 17 = 131072
2^ 18 = 262144
2^ 19 = 524288
2^ 20 = 1048576
2^ 21 = 2097152
2^ 22 = 4194304
2^ 23 =
?out of memory  error in 210
ready.
```

There is was intentional bug in the program, the main loop is infinite: 
line 140 jumps back to 120. However, the program does terminate. When `N=23` 
the recursion is so deep that the stack space is exhausted and we get and 
`out of memory error`.

Can we call it a "bug" when it was intentional?


### Factorial

The previous program was _too_ easy. 
It might give the impression that recursion is fully supported 
in C64 BASIC.

Let's step up the complexity: we want a subroutine that computes 
the value `N!` recursively. Recall that `!` denotes the _factorial_ 
operator defined as 

```
fac(0)= 1
fac(n)= fac(n-1)*n, if n>0
```

This differs very little from `POW2`, why is this harder?

Our subroutine will have an input `N`.
The subroutine will (recursively) compute `N!` and 
assign that to the global variable `R`.
Let us assume, as in `POW2`, that the signature includes _`N` is modified_.
And we recycle the `POW2` code as follows.

```basic 
200 rem inp:n; out:r=n!; mods:n
210 if n=0 then r=1:return
220 n=n-1
230 gosub 200
240 r=r*n
250 return
```

This does _not_ work. The problem is in line 240.
After the `GOSUB` on line 230, global variable `N` was modified (to an 
unspecified value), so the assignment `r=r*n` does not compute what we want.

We need to change the signature of `FAC(N)` to no longer use `N` as 
scratch variable, but to keep (retain) its value. That is the new spec 
of our subroutine, see line 200 below (`KEEP:N`). 

> This program is listed in lower case to make copy&paste to VICE easier.
> It is available as `2-FAC` on the [disk](recursion.d64).

```basic 
100 print "fac"
110 n=0
120 :gosub 200
130 :print n;chr$(157);"! =";r
140 :n=n+1
150 goto 120
160 :
200 rem inp:n; out:r=n!; keep:n
210 if n=0 then r=1:return
220 n=n-1
230 gosub 200
240 n=n+1:rem restore n
250 r=r*n
260 return
```

Fortunately, it is easy to implement the changed spec. 
For the case `N=0` (line 210), global variable `N` is not overwritten, so 
no need to update the code in that line.
For the case `N>0` (line 220-260), `N` is decremented on line 220.
Line 230 has the recursive call, _which now guarantees that the value 
of `N` is retained_. So we only need to add line 240, which increments `N` 
to restore its original value. This makes the assignment on line 250 
correct, and allows us to `RETURN` on line 260, because `N` and `R` now 
both adhere to the signature.

```
fac
 0! = 1
 1! = 1
 2! = 2
 3! = 6
 4! = 24
 5! = 120
 6! = 720
 7! = 5040
 8! = 40320
 9! = 362880
 10! = 3628800
 11! = 39916800
 12! = 479001600
 13! = 6.2270208e+09
 14! = 8.71782912e+10
 15! = 1.30767437e+12
 16! = 2.09227899e+13
 17! = 3.55687428e+14
 18! = 6.40237371e+15
 19! = 1.216451e+17
 20! = 2.43290201e+18
 21! = 5.10909422e+19
 22! = 1.12400073e+21
?out of memory  error in 210
ready.
```

`FAC(N)` has the same intentional bug as `POW2(N)`: the main loop is infinite.
Also this program terminates with an `out of memory error` due to a stack overflow.


### Fibonacci

In `FAC(N)` we learned that we sometimes need to retain values by restoring them.
Unfortunately it is not always possible to do so, easily, e.g. with a simple 
"repair expression" like `N=N+1`.

We will see this in our next toy example, Fibonacci, defined as follows.
See [wikipedia](https://en.wikipedia.org/wiki/Fibonacci_sequence) for details.

```
fib(0)= 0
fib(1)= 1
fib(n)= fib(n-1) + fib(n-2), if n>1
```

Our subroutine will have an input `N`.
The subroutine will (recursively) compute `FIB(N)` and 
assign that to the global variable `R`.
As before, we make sure that the value of `N` is retained.
We have two scratch variables `R0` and `R1`.

```basic 
200 rem inp:n; out:r=fib(n); keep:n; mods:r0,r1
210 if n<2 then r=n:return
220 n=n-1:gosub 200:r0=r
240 n=n-1:gosub 200:r1=r
250 r=r0+r1
260 n=n+2:rem restore n
270 return
```

The program above is a first attempt. Retaining the value of `N` is easy.
But the crux of this example is in `R0` and `R1`. Both are overwritten by the 
subroutine, so both are part of the mods-list. And when `R0` is 
on the mods-list, the second `GOSUB 200` (line 240) modifies `R0`, so line 
250 is incorrect. 

By the way, there is no `GOSUB 200` between `R1=R` and `R=R0+R1`, so that 
`R1` is on the mods-list is not a worry. We can easily see that by modifying 
out first attempt:

```basic 
200 rem inp:n; out:r=fib(n); keep:n; mods:r0
210 if n<2 then r=n:return
220 n=n-1:gosub 200:r0=r
240 n=n-1:gosub 200
250 r=r+r0
260 n=n+2:rem restore n
270 return
```

How do we solve that global variable `R0` is restored after the second 
subroutine call  (`GOSUB 200` on line 240)? That is not easy. `R0` is 
_computed_ so it is not a simple matter of restoring a decrement. 

The solution is to add a _stack_. In the program below, `S()` is the stack, 
and `S` is the stack pointer. Since arrays (`S()`) and scalars (`S`) are in 
a different name space (array versus scalar), we can use the same 
identifier (`S`). This is similar to having `S`, `S$` and `S%`. 
The stack is created on line 110 in the program below.

> This program is listed in lower case to make copy&paste to VICE easier.
> It is available as `3-FIB` on the [disk](recursion.d64).

```basic
100 print "fib"
110 dim s(30):s=0:rem setup stack
120 n=0
130 :t0=ti:gosub 200:t1=ti:t=(t1-t0)/60
140 :print "fib(";n;")=";r;
150 :f=1:if l>0 then f=t/l
160 :print int(t);chr$(157);"s(*";f;")"
170 :n=n+1:l=t
180 goto 130
200 rem inp:n; out:r=fib(n); keep:n,s
210 if n<2 then r=n:return
220 n=n-1:gosub 200
230 s(s)=r:s=s+1:rem push r
240 n=n-1:gosub 200
250 s=s-1:r=r+s(s):rem pop r
260 n=n+2:rem restore n
270 return
```

Line 210 is for the base cases (`N=0` and `N=1`); the base case leaves `N` and `S` unmodified.
Line 220 computes `FIB(n-1)` whose result `R` is _pushed_ on the stack (line 230).
Line 240 computes `FIB(n-2)` whose result `R` is incremented with the value popped from the stack (line 250).
This also restores the stack pointer `S`.
Line 260 restores `N`.

We have a main loop similar to the ones before, except that it is augmented 
with timing code. Line 130 calls the subroutine, but takes a timestamp before 
(`T0`) and after (`T1`) the call, to compute the execution time (`T`) in 
seconds (the `/60` converts jiffies to seconds).

Line 140 prints the function call and result.
Line 160 prints the execution time (`INT(T)`).
One extra feature is that line 150 computes a factor:
the ratio of the current execution time and the previous execution time.
The factor is also printed on line 160.

```
fib
fib( 0 )= 0  0s(* 1 )
fib( 1 )= 1  0s(* 1 )
fib( 2 )= 1  0s(* 3 )
fib( 3 )= 2  0s(* 1.66666667 )
fib( 4 )= 3  0s(* 2 )
fib( 5 )= 5  0s(* 1.7 )
fib( 6 )= 8  0s(* 1.64705882 )
fib( 7 )= 13  0s(* 1.67857143 )
fib( 8 )= 21  1s(* 1.65957447 )
fib( 9 )= 34  2s(* 1.64102564 )
fib( 10 )= 55  3s(* 1.625 )
fib( 11 )= 89  5s(* 1.625 )
fib( 12 )= 144  9s(* 1.62130178 )
fib( 13 )= 233  14s(* 1.62043796 )
fib( 14 )= 377  23s(* 1.6204955 )
fib( 15 )= 610  38s(* 1.61917999 )
fib( 16 )= 987  62s(* 1.61802575 )
fib( 17 )= 1597  101s(* 1.61856764 )
fib( 18 )= 2584  164s(* 1.61832186 )
fib( 19 )= 4181  266s(* 1.61802532 )
fib( 20 )= 6765  430s(* 1.61816247 )
fib( 21 )= 10946  697s(* 1.61813963 )
fib( 22 )= 17711  1128s(* 1.61823267 )

?out of memory  error in 250
ready.
```

`FIB(N)` has the same intentional bug as `POW2(N)` and `FAC(N)`: the main loop is infinite.
Also this program terminates with an `out of memory error` due to a stack overflow.

> It is worth noting that computing Fibonacci numbers using a recursive 
> algorithm is a bad idea. We can see that from the timing: `FIB(22)` takes 
> 1128 seconds which is nearly 20 minutes!
> 
> The reason for this long execution time is that computing `FIB(n)` recursively takes `2*FIB(N+1)-1` calls 
> (See [paper](https://courses.grainger.illinois.edu/cs374al1/fa2025/notes/03-dynprog.pdf)).
> Fibonacci numbers grow exponentially (see [wiki](https://en.wikipedia.org/wiki/Fibonacci_sequence#Computation_by_rounding)):
> `FIB(N) ~ 1.618^N / 2.236`, so the _computation time grows exponentially_ too.
> We see that back in the ratio printed by the BASIC programm, it matches 1.618.
> 
>   |  N    | 0 | 1 | 2 | 3 | 4 |  5 |  6 |  7 |  8 |   9 |  10 |  11 |  12 |  13 |   14 |   15 |   16 |   17 |   18 |    19 |    20 |    21 |    22 |
>   |:------|--:|--:|--:|--:|--:|---:|---:|---:|---:|----:|----:|----:|----:|----:|-----:|-----:|-----:|-----:|-----:|------:|------:|------:|------:|
>   | FIB   | 0 | 1 | 1 | 2 | 3 |  5 |  8 | 13 | 21 |  34 |  55 |  89 | 144 | 233 |  377 |  610 |  987 | 1597 | 2584 |  4181 |  6765 | 10946 | 17711 |
>   | calls | 1 | 1 | 3 | 5 | 9 | 15 | 25 | 41 | 67 | 109 | 177 | 287 | 465 | 753 | 1219 | 1973 | 3193 | 5167 | 8361 | 13529 | 21891 | 35421 | 57313 |
> 
> 
> It is much faster to use an iterative algorithm. Computation becomes linear in `N`. 
> It is even possible to compute `FIB(N)` in [logarithmic](https://en.wikipedia.org/wiki/Fibonacci_sequence#Matrix_form) time.


## Stack

The last toy example introduced a stack `S()`.
That was used to store intermediate values (`R0` in the Fibonacci example).
However all programs used a stack, the stack which is part of BASIC 
(which relies on the stack offered by the 6510 CPU).

It is good to know that BASIC uses the stack not only for `GOSUB`, but also 
for, for example, `FOR`-`NEXT`, for expression evaluation (`2*(3+4)`, and 
for interrupts (e.g. the keyboard scan).

The following program nests a couple of `FOR`-`NEXT` loops, and `GOSUB`s to 
see how many bytes they consume from the 6510 CPU. 

```basic 
100 print "initial stack"
110 gosub 900
120 :
200 print "stack use 'for'"
210 for i=0 to 0:gosub 900
220 for j=0 to 0:gosub 900
230 for k=0 to 0:gosub 900
240 :
300 print "'for's closed"
310 next i
320 gosub 900
330 :
400 print "stack use 'gosub'"
410 gosub 450:end
420 :
450 gosub 900
460 gosub 450
470 :
900 rem print stackpointer
910 poke 49152,186:rem tsx
920 poke 49153, 96:rem rts
930 sys  49152    :rem x=s
940 sa=peek(781)  :rem sa=s
950 print sa;     :rem print s
960 print sl-sa   :rem print delta
970 sl=sa:return  :rem sl=s "last"
```

- It should be noted that the 6510 stack is hardwired 
  to be in page 1 of the memory, that if from $0100 to $01FF.
  The high byte is fixed ($01), the low byte is determined by the CPU 
  register S. This register grows from $FF downwards.
  
- The subroutine starting at line 900 uses a small assembly program to 
  retrieve the 6510 stack pointer S and print it, together with the delta 
  to the previous value.
  
- Line 110 contains the first `GOSUB 900` printing the initial stack pointer 
  value (and a delta that does not make sense because there was no previous 
  stack pointer yet.

- The initial value appears to be 239; not exactly 255 ($FF) but close.
  Our BASIC program and the BASIC interpreter are running, so that 
  there are some values on the stack is not strange - but I don't know which.
  
```
initial stack
 239 -239
stack use 'for'
 221  18
 203  18
 185  18
'for's closed
 239 -54
stack use 'gosub'
 232  7
 225  7
 218  7
 211  7
 204  7
 197  7
 190  7
 183  7
 176  7
 169  7
 162  7
 155  7
 148  7
 141  7
 134  7
 127  7
 120  7
 113  7
 106  7
 99  7
 92  7
 85  7
 78
?out of memory  error in 960
ready.
```

- Next (lines 210, 220, and 230) come three (nested) `FOR`-`NEXT` loops.
  Each take a whopping 18 bytes of the stack. As [Mapping the C64](https://archive.org/details/Compute_s_Mapping_the_Commodore_64/page/n61/mode/2up)
  explains there is 1 byte for a tag (129), 2 bytes for a pointer to the loop variable, 
  5 bytes for the STEP value, 1 byte for the STEP sign, 5 bytes for the 
  TO value, and finally 2 bytes for the line number and 2 bytes for the address, 
  both the start statement of the FOR loop.
  
- A `NEXT` statement [cancels all inner loops](https://www.c64-wiki.com/wiki/FOR).
  This is what happens on line 310, so line 320 prints 239 again.
  
- The final test is on line 400, infinite recursion of the routine 450.
  A GOSUB uses 7 bytes ([C64-wiki](https://www.c64-wiki.com/wiki/Subroutine?utm_source=gemini)). 
  Again 1 for a tag (141), 2 bytes for the line number and 2 bytes for the 
  address, both the statement after the GOSUB, and finally 2 bytes for a JSR 
  overhead.
  
- Note that call 23 reduces raises the `OUT OF MEMORY ERROR`.
  This is the exact same value we found for the three toy examples.
  I do not fully understand why BASIC indicates the stak is empty 
  with a pointer at 78.
  
  [Mapping the C64](https://archive.org/details/Compute_s_Mapping_the_Commodore_64/page/n59/mode/2up)
  suggest that BASIC uses $0100-$010A of the stack area for floating point 
  conversions, and the KERNEL uses $0100-$013E for tape handling.
  This means the stack can grow down to $3F or 63. I'm guessing, BASIC needs
  78-63 = 15 bytes for other administrative purposes.
  
  If we look at `FIB(22)` we get the error `OUT OF MEMORY  ERROR IN 250` and 
  line 250 is _not_ a `GOSUB` but an assignment (`r=r+s(s)`). I suspect that 
  evaluating an expression uses stack space on top of the `GOSUB`
  
> **Conclusion** The C64 BASIC interpreter uses the 6510 stack for `GOSUB`s.
> It can only handle about 22 nested `GOSUB`s, which is insufficient for 
> the programs like the toy examples where the nesting depth is actually the 
> argument of a function.


## Real applications

The chapter [Theory](#theory) explains that for recursion in BASIC we need to 
_restore_ the value of global variables, maybe with the help of an additional 
stack (and array created by the programmer). The chapter also explains the 
BASIC uses the 6510 stack, which supports a maximum call depth of 22.

In this chapter we will show that real problems can still solved with 
recursion in BASIC.


### Towers of Hanoi


### 8 queens


### Expression parser


(end)

