# 12: Adidis Ababa

- _Points:_ 50 points
- _Version number_: b.03

## Password

```

```

## Manual

```
Lockitall                                            LOCKIT PRO r b.03
______________________________________________________________________

              User Manual: Lockitall LockIT Pro, rev b.03
______________________________________________________________________


OVERVIEW

    - We have verified passwords can not be too long.
    - Usernames are printed back to the user for verification.
    - This lock is attached the the LockIT Pro HSM-1.


DETAILS

    The LockIT Pro b.03  is the first of a new series  of locks. It is
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

    This is Software Revision 03. We have improved the security of the
    lock by ensuring passwords can not be too long.




(c) 2013 LOCKITALL                                            Page 1/1
```

## Writeup

Even though this level is b.03, it is significantly different from b.02 and b.04, and doesn't indicate any sort of continuity between them. Not sure what is the reason for this, or it might just be an overlooked detail in the CTF. Oh well.

Anyway, the key to note here is that there are several restrictions in place that prevent our typical buffer overflow attack from working:

1.  Only `0x13` bytes are allowed in the input, which is written to `0x2400` before getting copied to a buffer with `strcpy`.

    ```
    4454:  3e40 1300      mov    #0x13, r14
    4458:  3f40 0024      mov    #0x2400, r15
    445c:  b012 8c45      call   #0x458c <getsn>
    ```

2.  The buffer that is copied into is placed _after_ the stack addresses which contains the flag set from the `0x7D` interrupt called in `test_password_valid`, at `0x4210` (whereas the flag is at `0x420e`):

    ```
    448a:  8193 0000      tst   0x0(sp)
    448e:  0324           jz    #0x4496 <main+0x5e>
    4490:  b012 da44      call  #0x44da <unlock_door>
    4494:  053c           jmp   #0x44a0 <main+0x68>
    4496:  3012 1f45      push  #0x451f "That entry is not valid."
    449a:  b012 c845      call  #0x45c8 <printf>
    ```

    (_Note that `sp` is `0x420e` at `0x448a`_)

However, this level looks different in that it performs some `printf` calls to make the login screen more fancy, I suppose. The vulnerability here is a _format string vulnerability_, as the `printf` calls do not pass in a format string, allowing us to control the format string as well as the pointers to read from/write to, from the stack.

Since we know that function calls push arguments backwards on the stack, the first argument will be popped off the stack first. Based on the number of [format string parameters](https://www.owasp.org/index.php/Format_string_attack), this would pop off the same number of addresses off the stack to read from or write to.

We can use the `%x` format string parameter to read values from a fixed address, while `%n` allows us to write the number of characters to a particular address.

```c
// 0e422578256e
printf("%n%x\x0e\x42");
```
