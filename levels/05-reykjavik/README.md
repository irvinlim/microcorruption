# 05: Reykjavik

- _Points:_ 35 points
- _Version number_: a.03

## Password

Password in hex:

```
cff5
```

## Manual

```
Lockitall                                            LOCKIT PRO r a.03
______________________________________________________________________

              User Manual: Lockitall LockIT Pro, rev a.03
______________________________________________________________________


OVERVIEW

    - Lockitall developers  have implemented  military-grade on-device
      encryption to keep the password secure.
    - This lock is not attached to any hardware security module.


DETAILS

    The LockIT Pro a.03  is the first of a new series  of locks. It is
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

    This is Software Revision 02. This release contains military-grade
    encryption so users can be confident that the passwords they enter
    can not be read from memory.   We apologize for making it too easy
    for the password to be recovered on prior versions.  The engineers
    responsible have been sacked.




(c) 2013 LOCKITALL                                            Page 1/1
```

## Writeup

It looks like there is some shellcode within the stack, as can be seen in the instructions in main:

```
4438 <main>
4438:  3e40 2045      mov   #0x4520, r14
443c:  0f4e           mov   r14, r15
443e:  3e40 f800      mov   #0xf8, r14
4442:  3f40 0024      mov   #0x2400, r15
4446:  b012 8644      call  #0x4486 <enc>
444a:  b012 0024      call  #0x2400
444e:  0f43           clr   r15
```

Taking a look at `0x2400`, we can disassemble the following using the MSP430 assembler at <https://microcorruption.com/assembler>:

```
0b12 0412 0441 2452 3150 e0ff 3b40 2045
073c 1b53 8f11 0f12 0312 b012 6424 2152
6f4b 4f93 f623 3012 0a00 0312 b012 6424
2152 3012 1f00 3f40 dcff 0f54 0f12 2312
b012 6424 3150 0600 b490 cff5 dcff 0520
3012 7f00 b012 6424 2153 3150 2000 3441
3b41 3041 1e41 0200 0212 0f4e 8f10 024f
32d0 0080 b012 1000
```

```
0b12           push     r11
0412           push     r4
0441           mov      sp, r4
2452           add      #0x4, r4
3150 e0ff      add      #0xffe0, sp
3b40 2045      mov      #0x4520, r11
073c           jmp      $+0x10
1b53           inc      r11
8f11           sxt      r15
0f12           push     r15
0312           push     #0x0
b012 6424      call     #0x2464
2152           add      #0x4, sp
6f4b           mov.b    @r11, r15
4f93           tst.b    r15
f623           jnz      $-0x12
3012 0a00      push     #0xa
0312           push     #0x0
b012 6424      call     #0x2464
2152           add      #0x4, sp
3012 1f00      push     #0x1f
3f40 dcff      mov      #0xffdc, r15
0f54           add      r4, r15
0f12           push     r15
2312           push     #0x2
b012 6424      call     #0x2464
3150 0600      add      #0x6, sp
b490 cff5 dcff cmp      #0xf5cf, -0x24(r4)
0520           jnz      $+0xc
3012 7f00      push     #0x7f
b012 6424      call     #0x2464
2153           incd     sp
3150 2000      add      #0x20, sp
3441           pop      r4
3b41           pop      r11
3041           ret
1e41 0200      mov      0x2(sp), r14
0212           push     sr
0f4e           mov      r14, r15
8f10           swpb     r15
024f           mov      r15, sr
32d0 0080      bis      #0x8000, sr
b012 1000      call     #0x10
```

Putting everything else aside, let's put a breakpoint on the `cmp` instruction at `0x2448` as it looks like it could be comparing a hardcoded password value.

We see that the contents at `r4-0x24` contains the value of our flag. This means that we can change the control flow by making the first two bytes of the password `cf f5`.

Trying that out, it turns out that this unlocks the door successfully! This is because we did not execute the `jnz $+0xc` instruction, which then causes us to trigger an INT at `#0x2464` with `0x7f` pushed to the stack.

From the LockIT Pro manual:

> INT 0x7F.
> Interface with deadbolt to trigger an unlock if the password is correct.

This means that triggering an interrupt of type `0x7F` unlocks the deadbolt unconditionally.
