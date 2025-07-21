---
title: "InnoCTF 2019 Writeups"
description: "Writeups for InnoCTF 2019"
date: 2019-07-19
slug: "ctf-innoctf-2019"
taxonomies:
  tags: ["CTF"]
---

Writeups for [InnoCTF](http://innoctf.com/) 2019

<!-- more -->

## BG's

Can you check if this site is hiding something?

### Solution

`style.css` had a background property that lead to an image that contained the flag.

`InnoCTF{HVQMGcp00mN0MnmqwRNBExEBPFTMgnPQ}`

## RF

I was walking on rails when suddenly i found this on wooden fence: I3_nase7ncamco_r1cCt_t4T07_}Fnhs{1

### Solution

I looked at various ciphers and stumbled across the [Rail Fence cipher](https://en.wikipedia.org/wiki/Rail_fence_cipher). Then increasing the number of rails until it formed a flag.

```
I.............3............._.....
.n...........a.s...........e.7....
..n.........c...a.........m...c...
...o......._.....r.......1.....c..
....C.....t......._.....t.......4.
.....T...0.........7..._.........}
......F.n...........h.s...........
.......{.............1............
```

`InnoCTF{n0t_ca3sar_7h1s_t1me_7cc4}`

## Robots

I need your clothes your boots and your motorcycle

### Solution

`robots.txt` was in the root of the web server with the contents of:

```
Disallow: /*/super-secret-admin-panel
```

The super-secret-admin-panel then required sql injection of `' or 1=1 -- `.

## Call Me

### Solution

Using [Ghidra](https://ghidra-sre.org/) I replaced the existing printf function in main to point to `void flag()` address instead.

- Import the file as Raw Binary (this is because Ghidra messes up the ELF header if imported as a ELF binary)
- Find the memory address of the flag function

{{ img(src="call_me_flag.png", alt="Ghidra output of the flag function") }}

- Find the main function

{{ img(src="call_me_printf.png", alt="Ghidra output of the main function with a call to printf") }}

- Right click on the printf function and select patch instruction
- Change the call address to be the address of the flag function

{{ img(src="call_me_flag_patch.png", alt="Ghidra output of the printf call patched to the flag function") }}

- Export Program from Ghidra
- Run the executable

`InnoCTF{How_d1d_y0u_f1nd_m3_7f1bc88}`

## Quick Peek

### Solution

I discovered that this was a .NET executable, so I used [dnSpy](https://github.com/0xd4d/dnSpy) to run the executable with debugging.

{{ img(src="quick_peek.png", alt="dnSpy at a breakpoint in the flag function with the locals window showing the flag") }}

`InnoCTF{1337_SPAgh377i_CoD3}`
