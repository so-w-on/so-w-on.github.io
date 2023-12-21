# Introduction
This article will explain Stack-based buffer Overflow attacks in its simplest forms, i.e. when the attack is performed on a binary that doesn't have NX nor ASLR protections.

The absence of both protections is quite rare, but quite popular in entry-level binary exploitation ctf challenges.

# Attack Vector
This attack is possible when a program uses a vulnerable function to accept user input without input size check, and stores that input in memory.
When the user-input is stored into memory, it goes beyond the space allocated to the buffer and overwrites adjacent memory spaces.
When the EIP/RIP register gets overwritten, a segmentation fault occurs because the new value that the EIP/RIP register contains may not be a valid memory address that points to a valid memory space.
: By "vulnerable function" I mean functions that don't limit the number of characters read from use input. For example: `gets`, `scanf`, `strcpy`, `strcat`, `sprintf`, `vsprintf`,`snprintf`, `syslog`.

# A step-by-step of a stack-Buffer Overflow:
To see how this attack is performed, we will use a dummy vulnerable code and see the steps that lead to a buffer overflow.

## The code

## The Exploit

# Use Cases:
A buffer Overflow is very useful in diverting the program's normal execution flow.

Careful Calculations go into play when trying to make a buffer overflow useful, that is because if we overwrite the EIP/RIP register (which supposedly holds the next address to be executed) with random characters, we will run into a segmentation fault, since the new "address" in the EIP/RIP register doesn't point to anything readable and/or executable.

That is why we need to carefully craft our payload (user-input), so we can put the address of a certain pre-existing function that we want to run in the EIP/RIP, or we can make the EIP/RIP point to a shellcode we provide as part of the buffer.

![Desktop View](/assets/img/bufferoverflow/classicbof.png){: width="972" height="589"}

In this illustration we can see the first normal repartition of the stack that allocates some space to the buffer that will receive the user input, then the saved EBP followed by the saved EIP that should be pointing to the next instruction to execute.

With a buffer overflow attack, we exceed the space allocated to the buffer, overwrite the EBP and overwrite the EIP with the address pointing to the beginning of our shellcode.

The exact content of the shellcode depends on whether we want to redirect the execution flow to pre-existing functions, or to actually provide the instructions to execute ...

## Ret2win : Buffer Overflow to execute a, "win()", pre-existing function:
In the case of a Ret2win, we want our flow of execution to be redirected to a pre-existing win() function. Yes that is almost never the case in real life scenarios, but it is one of the basics that is encountered in ctf challenges and that focuses solely on the first step of "flow redirection".

The shellcode of a Ret2win would be a padding until the EIP, followed by the address of the win() function that will overwrite the EIP.
![Desktop View](/assets/img/bufferoverflow/ret2win.png){: width="972" height="589"}

The most used padding is just a controlled amount of the letter "A", for no particuler resaon.
But to obtain a "controlled amount", we need to know when to stop.

We can use gdb while disassembling the binary file to do so. 
1. Create a buffer of a certain amount of characters.
```bash
gdb$ pattern create 200
	> Output:
    > 'a_string_of_patterns'
```
then we run the binary and provide the string that was the output of the previous command, when the bianry asks to give input for our target buffer.

2. If a buffer overflow occurs, we should check the elements that are in EBP and find their offset:
```bash
gdb$ pattern offset fAA5a5
	> Output:
    > fAA5a5 found at offset: 86
```
So since teh offset of EBP is 86, the offset to EIP is 86+8= 94
The padding should be of length 94, followed by the address to the win function which can be found by disassembling the binary in gdb, since we are in a context where randomization isn't an issue.

The payload using python will look like this:
```python
from pwn import *
context.bits = 64

padding = b"A"*94
win_address = 0xthe_addr

shellcode = padding + p64(win_address)

p = process("./bin_file")
p.send(shellcode)
p.interactive()
```

## Buffer Overflow with instructions in the shellcode

# Shortcommings
This attack only works if both NX and ASLR are disabled, which is most probably never the case.
* When NX is enabled, but ASLR disabled, we can use the Ret2libc technique.
* When NX and ASLR are both enabled, we use the big guns: the ROP (Return-Oriented Programming) technique.