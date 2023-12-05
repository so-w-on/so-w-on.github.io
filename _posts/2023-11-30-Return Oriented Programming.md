Return-Oriented Programming is a technique that consists in using code pre-existing in the binary and chaining it together to form a desired chain of commands that will perform the desired instructions.
> As an example of pre-existing code, we have the "/bin/sh" command, the "system" function ...

## Gadgets
These chunks of "pre-existing code" are known as `gadgets`. A gadget, is code that exists in the program itself or in the libraries used by the program. It has the following form: `[instruction sequence]; ret`.
: These are gadget examples: `pop eax; ret`, `add eax, ebx; pop ebx; ret`, ...

To find usable gadgets along with their addresses, use the following:

```bash
$ ROPgadget --binary bin_file
```
or, to search for a more specific keyword (`rdi`, ...)

```bash
$ ROPgadget --binary bin_file | grep keyword
```

## Library gadget
Sometimes, we need some gadgets that aren't within the program itself, but that exist in the libraries it uses. In this article, we will talk about the C library (libc) and specifically about two gadgets `system` and `/bin/sh`, that can respectively execute any command passed to it and call a shell.
To find gadgets within the library , say `system` or `/bin/sh` for example, we us ethe following commands:

1. First, get the libraries used by the binary and their base addresses:
```bash
$ldd bin_file
	> Output:
	> <some libraries> <Their addresses>
	> libc.so.6 => /lib32/libc.so.6 (address of libc)
	> (*or /lib/x86_64-linux-gnu/libc.so.6 for 64-bit*)
```
2. Then get the offset from the libc base address of `system()`:
```bash
$readelf -s /lib32/libc.so.6 | grep system
	> Output:
	> number: offset_from_libc_base_@  <other......
```
3. Then get the offset from the libc base address of `/bin/sh`:
```bash
$strings -a -t x /lib32/libc.so.6 | grep /bin/sh
	> Output:
	> offset_from_libc_base_@ /bin/sh
```
## The ret instruction
This technique takes its name from the return instruction, so it is only fair we give some details about this one specific instruction.

Reminder:
These two registers are very important in the ROP technique.

RSP/ESP
: Stack Pointer register (Points to the top of the stack)

RIP/EIP
: Instruction Pointer register (Points to the next instruction to be executed)

When `ret` is executed, it pops the value pointed to by the stack pointer and places that value into the instruction pointer. In a very normal setting, the value popped off from the stack is an address that will be pointed to by the instruction pointer, meaning it is the address of the next instruction to be executed.

Here is an illustration to better visualize things:
TODO: Drawing

## Exploit outline
The payload used in executing a ROP exploit is constructed as follows:

### For a 32-bit binary file:

### For a 64-bit binary file:

## Ret2libc exploit:
This exploit is a ROP focused on functions within the library libc used by the binary file.
The payload used in executing a Ret2libc exploit is constructed as follows:
