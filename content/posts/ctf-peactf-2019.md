---
title: "peaCTF 2019 Writeups"
date: "2019-07-26T12:00:00-05:00"
tags: ["ctf"]
---

Writeups for [peaCTF](https://peactf.com/) 2019

<!--more-->

## RSA

Can you help Bob retrieve the two messages for a flag?

### Solution

The challenge included two files:

`auth_channel.txt`

```plaintext
Authenticated (unhashed) channel:
n = 59883006898206291499785811163190956754007806709157091648869
e = 65537
c = 23731413167627600089782741107678182917228038671345300608183
```

`enc_channel.txt`

```plaintext
Encrypted channel:
n = 165481207658568424313022356820498512502867488746572300093793
e = 65537
c = 150635433712900935381157860417761227624682377134647578768653
```

I had never done a RSA crypto CTF challenge before, so I read this great blog post [Attacking RSA for fun and CTF points](https://bitsdeep.com/posts/attacking-rsa-for-fun-and-ctf-points-part-1/).

#### enc_channel.txt

The `n` value in the `enc_channel.txt` file had [known factors](http://factordb.com/index.php?query=165481207658568424313022356820498512502867488746572300093793)

So we now have `p`, `q`, and `e`, I came across a [Crypto StackExchange question](https://crypto.stackexchange.com/questions/19444/rsa-given-q-p-and-e/19530) about decrypting `c` given `p`, `q`, `e`.

I attempted to use the python script in the answer but had difficulty getting it to work with our inputs. Eventually I converted the script to C# using the [BouncyCastle](https://www.nuget.org/packages/BouncyCastle/) NuGet package for the `BigInteger` and `modInverse` function. The following script was written in [LINQPad](https://www.linqpad.net/) for the nice dump functions.

```csharp
var p = new BigInteger("404796306518120759733507156677");
var q = new BigInteger("408801179738927870766525808109");
var e = new BigInteger("65537");
var c = new BigInteger("150635433712900935381157860417761227624682377134647578768653");

var n = p.Multiply(q);

var one = new BigInteger("1");
var phi = (p.Subtract(one).Multiply(q.Subtract(one)));

var d = e.ModInverse(phi);
d.Dump("d");

var result = c.ModPow(d, n);
result.Dump("Decoded");
result.ToString(16).Dump("hex");
```

Output:

```plaintext
d: 125212438875455843161244268921851095575914294443076546907249
Decoded: 8904929771347223901285886734450
hex: 7065614354467b663463743072
```

Converting the hex to ascii we get: `peaCTF{f4ct0r`

#### auth_channel.txt

The `n` value in the `auth_channel.txt` file had [known factors](http://factordb.com/index.php?query=59883006898206291499785811163190956754007806709157091648869)

I attempted to use the same script with the data from the `auth_channel.txt` file but ended up getting garbage hex out. After looking at a [BSidesSF 2018 CTF writeup](https://medium.com/@jmecom/bsidessf-2018-ctf-553ac34bd49f) I came across this section:

> RSA can be used for encryption and decryption:
> 
> - Encrypt a plaintext message *m* into ciphertext *c*: *c = m^e* mod *n*
> - Decrypt a ciphertext *c*: *m = c^d* mod *n*
> 
> It can also be used to produce and verify digital signatures:
> 
> - Produce a signature for a message *m*: *s = m^d* mod *n*
> - Verify a signature *s,* for *m*: *m == s^e* mod *n*

That gave me the idea to try verifying the signature instead of trying to decrypt the ciphertext. This script is very similar to the first one the only changes are the input and `var result = c.ModPow(e, n);` instead of `var result = c.ModPow(d, n);`

```csharp
var p = new BigInteger("192355607880290234740980693973");
var q = new BigInteger("311314068553039667905603427153");
var e = new BigInteger("65537");
var c = new BigInteger("23731413167627600089782741107678182917228038671345300608183");

var n = p.Multiply(q);

var one = new BigInteger("1");
var phi = (p.Subtract(one).Multiply(q.Subtract(one)));

var d = e.ModInverse(phi);
d.Dump("d");

var result = c.ModPow(e, n);
result.Dump("Decoded");
result.ToString(16).Dump("hex");
```

Output:

```plaintext
d: 42870301015457570297495318493842594795392515721010421474689
Decoded: 911845841250251271805
hex: 316e67317366756e7d
```

Converting the hex to ascii we get: `1ng1sfun}`

The final flag is `peaCTF{f4ct0r1ng1sfun}`

## Song of My People

A specific soundcloud rapper needs help getting into his password protected zipped file directory. The initial password is in the title. You just have to know your memes, and pick the right instrument! We were on the fence on giving you an image to go along with this puzzle, but the loincloth was too scandalous. Alternatively, you could bruteforce.

### Solution

The start of the challenge is a password protected zip file.

The challenge description lead me to [The Song of My People! meme](https://knowyourmeme.com/memes/the-song-of-my-people), which originates from an image of a Native American playing the violin. In the challenge description, it says "pick the right instrument", so the password ended up being `violin`.

The contents of the zip file were:

* `a lengthy issue.png`
  
  * This is a broken PNG image

* `Ice Cube - Check Yo Self Remix (Clean).mp3`
  
  * You better check yo self before you wreck yo self

* `README.txt`
```plaintext
one of the three files is a red herring, but a helpful one at that.
does any of this ADD up? This is a LONG problem.
```

#### a lengthy issue.png

Running `pngcheck` on the broken png file:

```plaintext
File: a lengthy issue.png (44525 bytes)
  chunk IHDR at offset 0x0000c, length 13
    1280 x 720 image, 8-bit palette, interlaced
  chunk sRGB at offset 0x00025, length 1
    rendering intent = perceptual
  chunk gAMA at offset 0x00032, length 4: 0.45455
  chunk PLTE at offset 0x00042, length 1212501072:  invalid number of entries (4.04167+08)
```

Opening `a lengthy issue.png` in HxD and finding the `PLTE` chunk:

![issue hxd](https://f001.backblazeb2.com/file/grbt-blog/images/posts/ctf-peactf-2019/song_issue_hxd.png)

Looking at the [libpng spec](http://www.libpng.org/pub/png/spec/1.2/PNG-Structure.html#Chunk-layout) and a [similar writeup](https://github.com/ctfs/write-ups-2015/tree/master/plaidctf-2015/forensics/png-uncorrupt). I figured out that the `HELP` text before the `PLTE` was the 4-byte length for the `PLTE` chunk.

Using the HxD data inspector I edited these 4 bytes to be the value of 453 in big-endian. The value for the PLTE header was the number of bytes between the end of PLTE and the beginning of the CRC for PLTE. The length also must be divisible by 3 for the PLTE chunk.

![issue hxd fixed](https://f001.backblazeb2.com/file/grbt-blog/images/posts/ctf-peactf-2019/song_issue_hxd_fixed.png)

The fixed image:

![issue](https://f001.backblazeb2.com/file/grbt-blog/images/posts/ctf-peactf-2019/song_issue.png)

#### Image hex

The hex in the image:

```plaintext
54 68 65 20 4c 69 62 72 61 72 79 20 6f 66 20 42 61 62 65 6c 3a 0a 28 77 69 74 68 20 6e
65 77 20 61 64 64 69 74 69 6f 6e 20 6f 66 20 61 6c 6c 20 74 68 65 20 70 6f 73 73 69 62 6c
65 20 64 69 73 73 20 74 72 61 63 6b 73 20 74 6f 20 65 76 65 72 20 62 65 20 6d 61 64 65
20 61 6e 64 20 65 76 65 72 20 63 6f 75 6c 64 20 62 65 20 6d 61 64 65 29
```

Decoding this to ascii results in:

> The Library of Babel: (with new addition of all the possible diss tracks to ever be made and ever could be made)

### Soundcloud

[Link to soundcloud](https://soundcloud.com/lil-redacted/live-concert-audio)

Soundcloud description:

> this concert is part of a larger tour that is archived completely in some kind of hexagonal library. The archive is named between "maybe" and a "repeat". Should be on the 371st page.
> 
> I would give you an mp3 of this audio, but I don't know how to navigate those sketchy websites.

The soundcloud "song" sounded like Morse code, after downloading the mp3 I discovered a [Morse code audio decoder](https://morsecode.scphillips.com/labs/audio-decoder-adaptive/).

The decoded text:

> SUP YALL ITS YA BOI LIL ICE CUBE MELTING OUT HERE IN THE HAWAII HEAT FOR ALL OF YOU. YOU GUESSED IT THIS IS LIVE AUDIO FROM MY WORLD TOUR. I REPEAT LIL ICE CUBES WORLD TOUR MAYBE A LIBRARY WILL HELP

#### Library of Babel

With the hint of [Library of Babel](https://libraryofbabel.info) I tried inputting the hex from the image as the `Hex Name` but that ended up being a dead end. I tried using the search for `lil ice cubes world tour`:

```plaintext
exact match:
Title: weif.rfgkd,zvjxxd Page: 371
Location: 2nhptm68h2sputfsdlbmi75s6qcfgu...-w2-s4-v08
```

Nice! The page number matches the soundcloud description.

I downloaded the [book](https://libraryofbabel.info/bookmark.cgi?weif.rfgkd,zvjxxd371:1) and counted the number of spaces in the book (48558), and tried the flag `{48_thousand_spaces_371}` but it was incorrect. The image said thousand_spaces or seats, so I searched how many times the word `seat` appears in book (3).

The final flag was `{3_thousand_spaces_371}`
