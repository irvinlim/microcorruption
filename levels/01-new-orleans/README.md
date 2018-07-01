# 01: New Orleans

- _Points:_ 10 points
- _Version number_: a.01

## Password

```
%=l9a7Y
```

## Manual

```
Lockitall                                            LOCKIT PRO r a.01
______________________________________________________________________

              User Manual: Lockitall LockIT Pro, rev a.01
______________________________________________________________________


OVERVIEW

    - This is the first LockIT Pro Lock.
    - This lock is not attached to any hardware security module.


DETAILS

    The LockIT Pro a.01  is the first of a new series  of locks. It is
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

    This is Software Revision 01.




(c) 2013 LOCKITALL                                            Page 1/1
```

## Writeup

As we learnt in the tutorial, the `check_password` function contains the instructions to determine if the entered password is correct or not. Let us take a look at the function:

```
44bc <check_password>
44bc:  0e43           clr   r14
44be:  0d4f           mov   r15, r13
44c0:  0d5e           add   r14, r13
44c2:  ee9d 0024      cmp.b @r13, 0x2400(r14)
44c6:  0520           jne   #0x44d2 <check_password+0x16>
44c8:  1e53           inc   r14
44ca:  3e92           cmp   #0x8, r14
44cc:  f823           jne   #0x44be <check_password+0x2>
44ce:  1f43           mov   #0x1, r15
44d0:  3041           ret
44d2:  0f43           clr   r15
44d4:  3041           ret
```

Since `r15` contains the address of our password on the stack, the `r13` register should contain the address of the byte currently being compared, since `r14` is incremented for each byte being compared. Also note that the `cmp.b` instruction has a `.b` suffix, which is defined as such in the instruction set manual:

> The suffix .B at the instruction mnemonic will result in a byte operation.

This means that `cmp.b` does a byte-wise comparison, instead of defaulting to word-wise (2 bytes) comparison with `cmp`.

In other words, the `cmp.b` instruction compares the value of the current letter with the value at `0x2400(r14)`. Since `r14` starts from `0` and increments by `1` when it moves on to the next letter, this means that the password should be stored at address `0x2400` on the stack.

Let's take a look at the stack when we are in the `check_password` function:

```
2400:   253d 6c39 6137 5900 0000 0000 0000 0000   %=l9a7Y.........
```

This gives us our password: `%=l9a7Y`
