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

# Case-By-Case ROP Use Cases
The following will be use cases of ROP depending on the protections that the binary has:

## ASLR with no NX:


# Exploit outline
The payload used in executing a ROP exploit is constructed as follows:

## For a 32-bit binary file:

## For a 64-bit binary file:

