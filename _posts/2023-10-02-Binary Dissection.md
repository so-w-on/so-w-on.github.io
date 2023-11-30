This article will discuss binary files' specificities that one may encounter. The article is structured as a walkthrough of the first few steps of a binary exploitation or reverse engineering.
The article will therefore provide useful commands that will help us dissect the binary and understand its format.

# Binary Typology
The most common types of binaries I came across are:

ELF binaries
: Executable & Linking Format. Used in Unix.

PE binaries
: Portable Executable. Used in Windows.

To define the binary type, we can use, the `file` command:

```bash
$ file binary
```

# Binary Protections

## NX
> In Windows, NX is refered to as `DEP` or Data Execution Prevention.
{: .prompt-info }

NX stands for `No eXecute`. The NX is a bit that when activated, prevents us from executing certain parts of Memory like Heap and stack. It also prevents from writting into certain parts of Memory such as the code section.
> What this entails when trying to exploit a binary is that it becomes impossible to just feed the exploit payload to the executable. Because whatever user input we pass, it will stay in a No eXecute zone and it will be considered as non-executable data.
But of course, researchers found a way to work around this: Return-Oriented Programming (You can check the 01/12/23 article).

## PIE
PIE stands fro `Position Independant Executable`. When enabled, this technique randomizes the base address everytime 

## ASLR
ASLR stands for `Address Space Layout Randomization`. This is a built-in technique in new systems. This technique randomizes the base address, helping against attacks such as Return Oriented Programming.

## Canary
This technique consists in writting random known values to areas susceptible to be targeted by buffer overflows. The OS checks for any alterations in the canary since this would indicate a possible buffer overflow which is another common attack.

To determine if these protections are enables in a file or not, one of the following commands is helpful:
1. using `checksec`
```bash
$ checksec binary
```
2. checking the file overview in `cutter`
```bash
$ cutter binary
```
3. Using `rabin2`
```bash
$ rabin2 binary
```