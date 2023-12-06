# Introduction
This article will explain Stack-based buffer Overflow attacks in its simplest forms, i.e. when the attack is performed on a binary that doesn't have NX nor ASLR protections.

The absence of both protections is quite rare, but quite popular in entry-level binary exploitation ctf challenges.

# Attack Vector
This attack is possible when a program uses a vulnerable function to accept user input without input size check.
: By "vulnerable function" I mean functions that don't limit the number of characters read from use input. For example: `gets`, `scanf`, `strcpy`, `strcat`, `sprintf`, `vsprintf`,`snprintf`, `syslog`.

# A step-by-step of a stack-Buffer Overflow:
To see how this attack is performed, we will use a dummy vulnerable code and see the steps that lead to a buffer overflow.

## The code

## The Exploit

# Use Cases:
A buffer Overflow is very useful in diverting the program's normal execution flow.
When the buffer overflows, it overwrites part of the stack, and with careful calculations, we can overwrite the EIP/RIP register which tells holds the next address to be executed. We can then put the address of a certain pre-existing function that we want to run in the EIP, or we can make the EIP point to a shellcode we provide as part of the buffer.

Below we can see both the payload (user-input) that lets us do these modifications and how that changes the stack and the code execution flow.

## Ret2win : Buffer Overflow to execute a win() pre-existing function:

## Classic Buffer Overflow : 

# Shortcomming
This attack only works if both NX and ASLR are disabled, which is most probably never the case.
* When NX is enabled, but ASLR disabled, we can use the Ret2libc technique.
* When NX and ASLR are both enabled, we use the big guns: the ROP (Return-Oriented Programming) technique.