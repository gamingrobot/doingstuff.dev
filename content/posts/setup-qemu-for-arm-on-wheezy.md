---
title: "Setup QEMU for ARM on Wheezy"
date: "2014-01-01T13:33:39-05:00"
highlightjslanguages: ["armasm"]
---

This is a quick little blog post for how to install QEMU for ARM on Debian 7.0 (Wheezy).
<!--more-->

---

## Installing Toolchains

Edit `/etc/apt/sources.list` and add

```
deb http://emdebian.org/debian/ stable main
deb http://ftp.us.debian.org/debian/ squeeze main
```

The squeeze repo is for the package `libgmp3c2` that is unavailable on wheezy.

Install the `emdebian` public key

```bash
apt-get install emdebian-archive-keyring
apt-get update
```

Install ARM toolchains

```bash
apt-get install linux-libc-dev-armel-cross libc6-armel-cross libc6-dev-armel-cross binutils-arm-linux-gnueabi gcc-4.4-arm-linux-gnueabi g++-4.4-arm-linux-gnueabi
```

---

### Installing QEMU

Install QEMU

```bash
apt-get install qemu qemu-user-static
```

Optional: Install QEMU GUI

```bash
apt-get install aqemu
```

---

### Hello ARM

Time to test everything is working.

`helloarm.s`

```armasm
.text
.globl _start
_start:
        mov   r0, #5         @ Load register r0 with the value 5
        mov   r1, #4         @ Load register r1 with the value 4
        add   r2, r1, r0     @ Add r0 and r1 and store in r2

stop:   b stop               @ Infinite loop to stop execution
```

Assemble, Link, Build

```bash
arm-linux-gnueabi-as -o helloarm.o helloarm.s
arm-linux-gnueabi-ld -o helloarm.elf helloarm.o
arm-linux-gnueabi-objcopy -O binary helloarm.elf helloarm.bin
```

Setup `flash.bin`

```bash
dd if=/dev/zero of=flash.bin bs=4096 count=4096
dd if=helloarm.bin of=flash.bin bs=4096 conv=notrunc
```

Launch QEMU

```bash
qemu-system-arm -M connex -pflash flash.bin -nographic -serial /dev/null
```

There should now be a `(qemu)` prompt, type `info registers` and should see something like this

```armasm
R00=00000005 R01=00000004 R02=00000009 R03=00000000
R04=00000000 R05=00000000 R06=00000000 R07=00000000
R08=00000000 R09=00000000 R10=00000000 R11=00000000
R12=00000000 R13=00000000 R14=00000000 R15=0000000c
PSR=400001d3 -Z-- A svc32
```

`R02=00000009` Register 2 has the value 9 in it so everything is working.

A more in-depth explanation of this helloarm program can be found [here](http://www.bravegnu.org/gnu-eprog/hello-arm.html).