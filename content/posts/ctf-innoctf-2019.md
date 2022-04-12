---
title: "InnoCTF 2019 Writeups"
date: "2019-07-19T12:00:00-05:00"
tags: ["ctf"]
---

Writeups for [InnoCTF](http://innoctf.com/) 2019

<!--more-->

## BG's

Can you check if this site is hiding something?

### Solution

`style.css` had a background property that lead to an image that contained the flag.

`InnoCTF{HVQMGcp00mN0MnmqwRNBExEBPFTMgnPQ}`

## RF

I was walking on rails when suddenly i found this on wooden fence: I3_nase7ncamсo_r1сCt_t4T07_}Fnhs{1

### Solution

I looked at various ciphers and stumbled across the [Rail Fence cipher](https://en.wikipedia.org/wiki/Rail_fence_cipher). Then increasing the number of rails until it formed a flag.

```plaintext
I.............3............._.....
.n...........a.s...........e.7....
..n.........c...a.........m...с...
...o......._.....r.......1.....с..
....C.....t......._.....t.......4.
.....T...0.........7..._.........}
......F.n...........h.s...........
.......{.............1............
```

`InnoCTF{n0t_ca3sar_7h1s_t1me_7сс4}`

## Robots

I need your clothes your boots and your motorcycle

### Solution

`robots.txt` was in the root of the web server with the contents of:

```plaintext
Disallow: /*/super-secret-admin-panel
```

The super-secret-admin-panel then required sql injection of `' or 1=1 -- `.

## Call Me

### Solution

Using [Ghidra](https://ghidra-sre.org/) I replaced the existing printf function in main to point to `void flag()` address instead.

* Import the file as Raw Binary (this is because Ghidra messes up the ELF header if imported as a ELF binary)
* Find the memory address of the flag function

![call_me_flag](/img/ctf-innoctf-2019/call_me_flag.png)

* Find the main function

![call_me_printf](/img/ctf-innoctf-2019/call_me_printf.png)

* Right click on the printf function and select patch instruction
* Change the call address to be the address of the flag function

![call_me_flag_patch](/img/ctf-innoctf-2019/call_me_flag_patch.png)

* Export Program from Ghidra
* Run the executable

`InnoCTF{How_d1d_y0u_f1nd_m3_7f1bc88}`

## Quick Peek

### Solution

I discovered that this was a .NET executable, so I used [dnSpy](https://github.com/0xd4d/dnSpy) to run the executable with debugging.

![quick_peek](/img/ctf-innoctf-2019/quick_peek.png)

`InnoCTF{1337_SPAgh377i_CoD3}`
