# 04: Cusco

- _Points:_ 25 points
- _Version number_: b.02

## Password

```
414141414141414141414141414141414644
```

_ASCII: `AAAAAAAAAAAAAAAAFD`_

## Manual

```
Lockitall                                            LOCKIT PRO r b.02
______________________________________________________________________

              User Manual: Lockitall LockIT Pro, rev b.02
______________________________________________________________________


OVERVIEW

    - We have fixed issues with passwords which may be too long.
    - This lock is attached the the LockIT Pro HSM-1.


DETAILS

    The LockIT Pro b.02  is the first of a new series  of locks. It is
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

    This is Software Revision 02. We have improved the security of the
    lock by  removing a conditional  flag that could  accidentally get
    set by passwords that were too long.




(c) 2013 LOCKITALL                                            Page 1/1
```

## Writeup

This time, it looks like the LockITAll makers have learnt from their mistake and removed the unnecessary condition check in `login`. The rest of the instructions looks pretty similar as well.

However, on closer inspection, we notice several differences. Firstly, the location on the stack where our input is located is at `0x43ed`, and allows an input size of `0x30` bytes. The key here is that only 16 bytes was allocated for the buffer, which means that we have our classic buffer overflow attack.

The following is the stack right after entering our password value `AAAA`:

```
43e0:   5645 0000 a045 0200 ee43 3000 1e45 4141   VE...E...C0..EAA
43f0:   4141 0000 0000 0000 0000 0000 0000 3c44   AA............<D
```

Notice at `0x43fd` it contains `0x443c` (little-endian), which happens to be the return address for the login function. We can verify this by setting a breakpoint right on the `ret` instruction at the end of `login`.

This means that we can modify the control flow of execution by overriding the address to jump to at `0x43fe`! Since `unlock_door` is located `0x4446`, we need to overflow the buffer with `46 44` after 16 bytes of gibberish.

This gives us a password of: `AAAAAAAAAAAAAAAAFD`
