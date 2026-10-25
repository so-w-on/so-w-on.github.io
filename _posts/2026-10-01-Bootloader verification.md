---
title: Bootloader verification
author: Mouna
date: 2026-10-01 11:00:00 +0800
categories: [1 Project-- PQC SecureBoot]
---
# Introduction
A reliable bootloader verification is one of the basics of establishing trust in an embedded product's integrity and authenticity. In this post we explore the structure of a bootloader that verifies a sort of Magic value, the application size and version, as well as the calculated hash (to ensure integrity), and lastly a signature to check the authenticity of the verifications.
The algorithms used will be swapped for more Post-quantum robust algorithms in a later post. The goal would be to benchmark and compare the overhead in matters of resource consumption.

The setup uses an STM32 NUCLEO-F446RE board.

The initial setup can be done following ControllersTech tutorial here (Part1-3):

Source: <https://controllerstech.com/category/stm32/stm32-custom-bootloader-ota/>
{: .prompt-warning }

The tutorial uses crc32 for verification, which we swap for sha256 from tinycrypt.

# Tutorial Summary
After going through the tutorial, we are left with a structure as can be seen below:
A bootloader that performs a series of checks for sanity, correctness and integrity through a crc32 check, then jumps to the application.
We will be building on this squeletton so if it's not so clear, the ControllersTech tutorials 1 to 3 covers it pretty well.

In the next steps, we will switch out the crc32 for a sha256 function. Then add a signature to ensure authenticity of the header.
(In the next blog, we will go beyond classical algorithms and explore the possibility of using PQC-robust ones.)

![Desktop View](/assets/img/bootloader/BootLogicCRC.png){: width="972" height="589"}

# SHA256
The logic as was defined in the tutorial makes it quite easy to swap out the crc32 for sha256.
1. First, we will modify the application header so that it contains a SHA256 hash and a signature (for the next step).
    ```C
        typedef struct
        {
            uint32_t magic;
            uint32_t version; /* Image version for anti-rollback */
            uint32_t size; /* Application size in bytes */
            uint8_t hash[32]; /* SHA-256 of the application */
            uint8_t signature[64]; /* signs the previous bytes of the header */
        } app_header_t;
    ```

2. We will change the header_fill to calculate the sha256 digest instead of crc32 for the application integrity, which will then be added to the header of the application.

    The header_fill file stays pretty much the same aside from the swap to sha256 in the ahshing logic (Check the file in the github repo for more implementation specification)
    ![Desktop View](/assets/img/bootloader/hash256.png){: width="972" height="589"}

3. We will swap out the crc32.C file for Tinycrypt source files for sha256. These files will be included in the bootloader project to calculate the sha256 as part of the bootloader verification of the application.

    First, get the TinyCrypt files from their official repository:

    Source: <https://github.com/intel/tinycrypt/tree/master>
    {: .prompt-warning }

    We will need the sha256.c sha256.h utils.c utils.h constants.h files divided into two folders inside the Bootloader/Core/Inc/tinycrypt and Bootloader/Core/Src/tinycrypt

# Signature