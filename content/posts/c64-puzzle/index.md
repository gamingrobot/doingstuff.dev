+++
title = "Commodore 64 Puzzle"
description = "A fun Commodore 64 Puzzle"
date = 2020-02-27
slug = "c64-puzzle"
[extra]
image = "problem.jpg"
+++

My college professor posted this image with the question

> What is the output when enter is pressed?

{{ img(src="problem.jpg", align="center" alt="Commodore 64 screen with a BASIC program") }}

<!-- more -->

I recognized this as a [Commodore 64](https://en.wikipedia.org/wiki/Commodore_64) and decided to dive deeper into what happens when `SYS 49152` is run.

## Some Basics

`PEEK(<address>)` is used to look at the memory for the specified address.

`POKE <address>,<value>` writes the value to the specified address.

## The Main Program

We can start by loading the program into a [Commodore 64 Emulator](http://vice-emu.sourceforge.net/).

```
10 X=49152
15 PRINT X
20 Y=11
30 FOR M=0 TO Y
40 PRINT PEEK(X+M);
50 PRINT " ";
60 NEXT M
```

{{ img(src="vice-screen-20200229000050.png", alt="Commodore 64 screen with bad output from the main program") }}

The output doesn't match the output from the original picture. According to the [C64 wiki](https://www.c64-wiki.com/), the `49152` address is the location of the [MONITOR$C000](https://www.c64-wiki.com/wiki/MONITOR$C000,_MONITOR$8000_(Commodore)). So the BASIC program is printing the bytes of the machine code program located in MONITOR$C000.

## Matching the Contents

We can attempt to match the contents of the `49152-49164` memory block by manually poking bytes into memory.

```
10 POKE 49152,169
11 POKE 49153,83
12 POKE 49154,141
13 POKE 49155,244
14 POKE 49156,5
15 POKE 49157,169
16 POKE 49158,2
17 POKE 49159,141
18 POKE 49160,244
19 POKE 49161,217
20 POKE 49162,96
21 POKE 49163,0
```

{{ img(src="vice-screen-20200229001253.png", alt="Commodore 64 screen with matching output from the original program") }}

## Success?

Now when running `SYS 49152` we get a nice red heart.

{{ img(src="vice-screen-20200229001432.png", alt="Commodore 64 screen with a red heart in the middle of the screen") }}

## But Why?

How does `169 83 141 244 5 169 2 141 244 217 96 0` result in a red heart on the screen?

The bytes are machine code for the Commodore 64. So we can look up the opcodes to convert them to assembly.

### Decoding

Converting the bytes into hex:

`A9 53 8D F4 05 A9 02 8D F4 D9 60 00`

Using the [Opcode lookup table](https://www.c64-wiki.com/wiki/Opcode) we can decode the bytes:

- A9: [LDA #nn](https://www.c64-wiki.com/wiki/LDA) (**L**oa**D** **A**ccumulator)
- 53(83 dec): Value loaded into the [accumulator](https://www.c64-wiki.com/wiki/Accumulator)
- 8D: [STA nnnn](https://www.c64-wiki.com/wiki/STA) (**ST**ore **A**ccumulator)
- 05F4(1524 dec): Address to store the value from the accumulator
- A9: [LDA #nn](https://www.c64-wiki.com/wiki/LDA)
- 02(2 dec):  Value loaded into the accumulator
- 8D: [STA nnnn](https://www.c64-wiki.com/wiki/STA)
- D9F4(55796 dec): Address to store the value from the accumulator
- 60: [RTS](https://www.c64-wiki.com/wiki/RTS) (**R**e**T**urn from **S**ubroutine)


Here is the full assembly code:

```
LDA #53
STA 05F4
LDA #02
STA D9F4
RTS
```

If we convert this to a BASIC program it would be:

```
10 POKE 1524,83
20 POKE 55796,2
```

### What is it doing?

So we have figured out that the machine code is writing two values to two different memory addresses.

`POKE 1524,83`

The `1524` address is used by the C64 for [Screen RAM](https://www.c64-wiki.com/wiki/Screen_RAM) allowing you to write characters directly to the screen. The value `83` is the character code for the heart in the [C64's character set](https://www.c64-wiki.com/wiki/Character_set).

`POKE 55796,2`

The `55796` address is used to specify the color of the character on the screen using [Color RAM](https://www.c64-wiki.com/wiki/Color_RAM). The value `2` is red on the [color lookup table](https://www.c64-wiki.com/wiki/Color).

For more information on the C64's memory map you can look [here](https://www.c64-wiki.com/wiki/Memory_Map).

### Having Some Fun

Now that we understand how this works we can write our own colorful characters to the screen.

{{ img(src="vice-screen-20200229004946.png", alt="Commodore 64 screen with a red heart, a teal heart, and a green spade") }}
