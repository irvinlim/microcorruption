# 03: Sydney

- _Points:_ 15 points
- _Version number_: a.02

## Password

```
2251486f2169647e
```

_ASCII: `"QHo!id~`_

## Manual

```
Lockitall                                            LOCKIT PRO r a.02
______________________________________________________________________

              User Manual: Lockitall LockIT Pro, rev a.02
______________________________________________________________________


OVERVIEW

    - We have revised the software in revision 02.
    - This lock is not attached to any hardware security module.


DETAILS

    The LockIT Pro a.02  is the first of a new series  of locks. It is
    controlled by a  MSP430 microcontroller, and is  the most advanced
    MCU-controlled lock available on the  market. The MSP430 is a very
    low-power device which allows the LockIT  Pro to run in almost any
    environment.

    The  LockIT  Pro   contains  a  Bluetooth  chip   allowing  it  to
    communiciate with the  LockIT Pro App, allowing the  LockIT Pro to
    be inaccessable from the exterior of the building.

    There is  no default password  on the LockIT  Pro---upon receiving
    the LockIT Pro, a new password must be set by connecting it to the
    LockIT Pro  App and  entering a password  when prompted,  and then
    restarting the LockIT Pro using the red button on the back.

    This is Hardware  Version A.  It contains  the Bluetooth connector
    built in, and one available port  to which the LockIT Pro Deadbolt
    should be connected.

    This is  Software Revision 02.  We have received reports  that the
    prior  version of  the  lock was  bypassable  without knowing  the
    password. We have fixed this and removed the password from memory.




(c) 2013 LOCKITALL                                            Page 1/1
```

## Writeup

This time, the lock doesn't seem to have the password stored on the stack in memory. Instead, let's look at the `check_password` function once again:

```
448a <check_password>
448a:  bf90 2251 0000 cmp #0x5122, 0x0(r15)
4490:  0d20           jnz $+0x1c
4492:  bf90 486f 0200 cmp #0x6f48, 0x2(r15)
4498:  0920           jnz $+0x14
449a:  bf90 2169 0400 cmp #0x6921, 0x4(r15)
44a0:  0520           jne #0x44ac <check_password+0x22>
44a2:  1e43           mov #0x1, r14
44a4:  bf90 647e 0600 cmp #0x7e64, 0x6(r15)
44aa:  0124           jeq #0x44ae <check_password+0x24>
44ac:  0e43           clr r14
44ae:  0f4e           mov r14, r15
44b0:  3041           ret
```

The `r15` register contains the address of the password we entered, so what these instructions are doing is to compare every dword (2 bytes) stored in `r15` with an immediate value. These are the values that make up the actual password!

Since the CPU is little-endian, it means that when we see `#0x5122` in the instruction, the byte ordering is actually `0x22 0x51` in memory, from a low to high address. In other words, when we are comparing two bytes at `0x0(r15)`, we compare `0x22` with `0x0(r15)` and `0x51` with `0x1(r15)`.

As such, we can simply take all of the immediate values in the `cmp` instructions, and pass the following through our favourite hex to ASCII converter:

```
2251486f2169647e
```

This gives us our password: `"QHo!id~`
