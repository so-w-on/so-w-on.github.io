# Introduction
A Simple Buffer Overflow is useful when The NX and ASLR protections aren't activated. This is pretty much only possible in CTF pwn challenges. These protections have become much more commun, which is why Binary Experts have found new ways to bypass them. ROP or Return-Oriented Programming is one of these techniques.

Return-Oriented Programming is a technique that consists in using code pre-existing in the binary and chaining it together to form a desired chain of commands that will perform the desired instructions.

# Gadgets
These chunks of "pre-existing code" are known as `gadgets`. A gadget, is code that exists in the program itself or in the libraries used by the program. It has the following form: `[instruction sequence]; ret`.
: These are gadget examples: `pop eax; ret`, `add eax, ebx; pop ebx; ret`, ...

To find usable gadgets in the binary file along with their addresses, use the following:

```bash
$ ROPgadget --binary bin_file
```
or, to search for a more specific keyword (`rdi`, ...)

```bash
$ ROPgadget --binary bin_file | grep keyword
```

## ROP execution outline
First of all, we will check the sections of the binary to determine which can be executed.
The sections that can be executed are of interest to us because if they contain gadgets, these can be executed. So we need to search for the gadgets to use in these sections.

To get these sections, we run the following command:
```bash
$ readelf -S bin_file
```
This will output information about all the headers of the binary. The ones that may be of interest to us are the following:
* .plt : Procedural Linkage Table : permits dynamic linking of the binary to functions within the libc.
* .text : contains the code of the binary.
* .got : Global Offsets Table : contains the addresses of the functions linked in plt.
* .bss : contains static data.
* .data : contains variable data