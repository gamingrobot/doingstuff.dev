+++
title = "Embedding Version Info into ELF Shared Libraries"
date = 2023-03-22
slug = "elf-shared-library-version"
[taxonomies]
tags = ["Linux"]
+++

*How to embed version information into an ELF shared library?*

That is the question I ended up answering for myself, and here are the methods I discovered for embedding version information.

<!-- more -->

This will cover embedding a product/release version in libraries distributed outside a package. Packages have their own version that can be referenced, so we don't need to embed a version.

## What about ABI version?

ABI version can be separate from your product/release version. If you haven't changed the ABI interface, you can keep the same ABI version while updating the product/release version on new builds. There are some great [resources](https://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html), [policy](https://gcc.gnu.org/onlinedocs/libstdc++/manual/abi.html#abi.changes_allowed), and [tools](https://www.gnu.org/software/libtool/manual/libtool.html#Versioning) for how to handle ABI versioning.

{% info() %}
Symbol versioning is another way to handle ABI changes without changing the ABI version, this blog post is [all about symbol versioning](https://maskray.me/blog/2020-11-26-all-about-symbol-versioning). 
{% end %}

## Ways of embedding version information

Okay, so you want to embed a product/release version in your ELF shared libraries. How should you do it?

### SCCS String

Probably the easiest to add, this method uses the [Source Code Control System](https://en.wikipedia.org/wiki/Source_Code_Control_System) `@(#)` convention to embed the version. 

```c
#define VERSION "1.2.3"

static char sccsid[] __attribute__((used)) = "@(#)Version " VERSION;
```

Then the version can be retrieved via [what(1)](https://man7.org/linux/man-pages/man1/what.1p.html) or `strings libfoo.so | grep "@(#)"`

```console
$ strings libfoo.so | grep "@(#)"
@(#)Version 1.2.3
```

This is human readable but slow since it has to look at all the strings in the binary. If you want to get the version programmatically, we have another method.

### ELF Note Header

This method creates an ELF note header by defining a note struct with the descriptor field. Then uses `__attribute__((used, section(...), aligned(4)))` to tell the compiler to add the ELF header.

```c
#include <elf.h>

#define VERSION "1.2.3"
#define NOTE_SECTION ".note.foo.version"

//Elf64_Nhdr + desc
struct version_note {
    Elf64_Word	namesz;
    Elf64_Word	descsz;
    Elf64_Word	type;
    char    desc[sizeof(VERSION)];
};

__attribute__((used, section(NOTE_SECTION), aligned(4)))
static const struct version_note version = {
    .namesz = 0,
    .descsz = sizeof(VERSION),
    .type = NT_VERSION,
    .desc = VERSION
};
```

> Note sections contain a series of notes. Each note is followed by the name field (whose length is defined in `namesz`) then by the descriptor field (whose length is defined in `descsz`) and whose starting address has a 4 byte alignment. Neither field is defined in the note struct due to their arbitrary lengths.

{% info() %}
More details in the [elf manpage](https://man7.org/linux/man-pages/man5/elf.5.html) under the `Notes (Nhdr)` section.
{% end %}

Then `readelf -Wn` or an ELF parsing library can read the version header.

```console
$ readelf -Wn libfoo.so
Displaying notes found in: .note.foo.version
  Owner                 Data size       Description
  (NONE)               0x00000006       NT_VERSION (version)       description data: 31 2e 32 2e 33 00
```

The downside is the output is in hex when using `readelf`, unlike the human readable format of SCCS above. But this method is faster since it's only reading the ELF headers rather than the whole binary.
