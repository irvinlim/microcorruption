# 03: Hanoi

- _Points:_ 20 points
- _Version number_: b.01

## Password

```
AAAAAAAAAAAAAAAAL
```

Any password with the 17th character being `L` will work.

## Manual

```
Lockitall                                            LOCKIT PRO r b.01
______________________________________________________________________

              User Manual: Lockitall LockIT Pro, rev b.01
______________________________________________________________________


OVERVIEW

    - This lock is attached the the LockIT Pro HSM-1.
    - We have updated  the lock firmware  to connect with the hardware
      security module.


DETAILS

    The LockIT Pro b.01  is the first of a new series  of locks. It is
    controlled by a  MSP430 microcontroller, and is  the most advanced
    MCU-controlled lock available on the  market. The MSP430 is a very
    low-power device which allows the LockIT  Pro to run in almost any
    environment.

    The  LockIT  Pro   contains  a  Bluetooth  chip   allowing  it  to
    communiciate with the  LockIT Pro App, allowing the  LockIT Pro to
    be inaccessable from the exterior of the building.

    There  is no  default  password  on the  LockIT  Pro HSM-1.   Upon
    receiving the  LockIT Pro,  a new  password must  be set  by first
    connecting the LockitPRO HSM to  output port two, connecting it to
    the LockIT Pro App, and entering a new password when prompted, and
    then restarting the LockIT Pro using the red button on the back.

    LockIT Pro Hardware  Security Module 1 stores  the login password,
    ensuring users  can not access  the password through  other means.
    The LockIT Pro  can send the LockIT Pro HSM-1  a password, and the
    HSM will  return if the password  is correct by setting  a flag in
    memory.

    This is Hardware  Version B.  It contains  the Bluetooth connector
    built in, and two available  ports: the LockIT Pro Deadbolt should
    be  connected to  port  1,  and the  LockIT  Pro  HSM-1 should  be
    connected to port 2.

    This is Software Revision 01,  allowing it to communicate with the
    LockIT Pro HSM-1




(c) 2013 LOCKITALL                                            Page 1/1
```

## Writeup

For starters, let's look at the instructions at `test_password_valid`:

```
4454 <test_password_valid>
4454:  0412           push  r4
4456:  0441           mov   sp, r4          # Load sp into r4
4458:  2453           incd  r4              # r4 points to 2 bytes after sp (r4 points to 43fc)
445a:  2183           decd  sp              # Move stack pointer upwards 2 bytes (43f8), r4 points to 4 bytes after sp
445c:  c443 fcff      mov.b #0x0, -0x4(r4)  # Load 0x0 into 43f8
4460:  3e40 fcff      mov   #0xfffc, r14
4464:  0e54           add   r4, r14         # Subtract r14 by 0x4, r14 is equal to sp
4466:  0e12           push  r14
4468:  0f12           push  r15
446a:  3012 7d00      push  #0x7d
446e:  b012 7a45      call  #0x457a <INT>
4472:  5f44 fcff      mov.b -0x4(r4), r15   # Move &43f8 to r15
4476:  8f11           sxt   r15             # Sign extend r15 from 8 to 16 bits
4478:  3152           add   #0x8, sp
447a:  3441           pop   r4
447c:  3041           ret
```

Before `INT`:

```
pc  446e  sp  43f2  sr  0001  cg  0000
r04 43fc  r05 5a08  r06 0000  r07 0000
r08 0000  r09 0000  r10 0000  r11 0000
r12 0000  r13 0000  r14 43f8  r15 2400
```

After `INT`:

```
pc  4472  sp  43f2  sr  0001  cg  0000
r04 43fc  r05 5a08  r06 0000  r07 0000
r08 0000  r09 0000  r10 0000  r11 0000
r12 0000  r13 0000  r14 007d  r15 7d00
```

It seems that `test_password_valid` is not affected by the password value we entered (which is located at `0x2400`) at all... Let's try fuzzing instead.

Entering 1024 `A`s into the password field, we see that the value is truncated to 28 bytes. This is supported by the `puts` call with a argument of `0x1c` in `login`:

```
4530:  b012 de45      call  #0x45de <puts>
4534:  3e40 1c00      mov   #0x1c, r14
4538:  3f40 0024      mov   #0x2400, r15
453c:  b012 ce45      call  #0x45ce <getsn>
4540:  3f40 0024      mov   #0x2400, r15
4544:  b012 5444      call  #0x4454 <test_password_valid>
```

Hmm... But the I/O console displayed a message: `Remember: passwords are between 8 and 16 characters.`, this might indicate that it's possible to change the execution flow if the password is outside of that range!

Since `test_password_valid` yields no more clues, let's take a look at `login` instead:

```
4544:  b012 5444      call  #0x4454 <test_password_valid>
4548:  0f93           tst   r15
454a:  0324           jz    $+0x8
454c:  f240 1900 1024 mov.b #0x19, &0x2410
4552:  3f40 d344      mov   #0x44d3 "Testing if password is valid.", r15
4556:  b012 de45      call  #0x45de <puts>
455a:  f290 4c00 1024 cmp.b #0x4c, &0x2410
4560:  0720           jne   #0x4570 <login+0x50>
4562:  3f40 f144      mov   #0x44f1 "Access granted.", r15
4566:  b012 de45      call  #0x45de <puts>
456a:  b012 4844      call  #0x4448 <unlock_door>
456e:  3041           ret
4570:  3f40 0145      mov   #0x4501 "That password is not correct.", r15
4574:  b012 de45      call  #0x45de <puts>
```

The `cmp.b` instruction at `455a` looks interesting... It compares the value of `&0x2410` with `#0x4c`, but seemingly for no reason. We remember that our input is stored at `0x2400` on the stack, but since `puts` allows us to insert 28 bytes of input, we can actually insert a byte at the 17th character (which corresponds to `0x2410`)!

We know that `0x4c` corresponds to ASCII `L`, so any password whose 17th character is `L` would work. We can use the following password to unlock the door: `AAAAAAAAAAAAAAAAL`
