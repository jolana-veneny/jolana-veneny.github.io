---
layout: post
tag: WriteUps
author: JLN
---
## Overflow Exploit: Just Do It
This slightly more complicated overflow challenge is available [here](https://github.com/hoppersroppers/nightmare/tree/master/modules/04-Overflows/04-bof_variable/tw17_justdoit) as part of the Nightmare RE course. I used Python and the pwntool module to write the exploit.

For this challenge to work, you need to supply the flag.txt in the same directory as the binary.

As always, let's start with trying to understand what this binary is and what it does. When we run it with the `file` command we can see that it is a 32-bit executable. 

<img src="https://user-images.githubusercontent.com/101567957/190863737-a1ae23bc-cd95-4685-8b20-4200986c5e8c.png" width="700">

The next step is to actually run the binary. 

<img src="https://user-images.githubusercontent.com/101567957/190866162-414ef6a7-9af2-4ca1-93df-e3f77c3feb41.png" width="700">

Not entirely surprising, the binary requires a password to authenticate and this is the time when we open this binary in Ghidra:
This is what the main function looks like in the decompiler:

<img src="https://user-images.githubusercontent.com/101567957/190867392-b5f7fcd7-3437-4412-b52d-1b14f8646f5d.png" width="700">

Look at this piece of code at the end: 
```
  iVar2 = strcmp(vulnBuf,PASSWORD);
  if (iVar2 == 0) {
    target = success_message;
  }
```
This means that if the vulnBuf value (a value we input for the password) and PASSWORD are equal, the binary will log us in.
When you go to the address where PASSWORD is stored at 0804a03c, we can see that the password is ```P@SSW0RD```.

<img src="https://user-images.githubusercontent.com/101567957/190867587-786d50df-1f63-4a23-b703-5ca1bab21af2.png" width="700">

So, let's put together a Python script to run the binary and enter the password:
```
from pwn import process, p32

target = process('just_do_it')
print(target.recvuntil(b"password.\n"))
target.sendline(b"P@SSW0RD\x00")
target.interactive()
```
As a note, the password we are submitting through Python is actually the password plus a new line character("\x00").
So we submit the password by running our password.py code:

<img src="https://user-images.githubusercontent.com/101567957/190870411-24c811c3-680b-4de8-9869-46a98d9e35ca.png" width="700">

Our password has been accepted and... nothing. This doesn't solve our problem and doesn't print the flag. 
(I mean, this is an overflow challenge, it makes perfect sense)

This is the part where we need to start looking at the variables, their addresses, and overflows.
Looking at the offsets we can see that the `vulnBuf` variable into which our password response gets written, is at the bottom of the stack and we could use an overflow to overwrite other variables.

<img src="https://user-images.githubusercontent.com/101567957/190915332-bba77590-0d9a-4591-800d-547497311988.png" width="700">

When you scroll back up to the decompiled C code you will see that the next variable on the stack is `local_18`, the flag variable which loads up the content of the flag file. Overwriting this variable will not help us much. But when you look at the nexdt variable `target` in the C code you will see that at the very end, `target` gets printed with `puts`. So, what if we changed the `target` to the address where the flag function is stored?
If that doesn't make sense try following this diagram I drew:

![flow](https://user-images.githubusercontent.com/101567957/190915383-5b372bc9-5827-4f51-85ca-0a41712f11e0.jpg)

In the upper corner you can see the logic of the flow - overflowing the `vulnBuf` variable and spill into `target`.
On the left you will see how this changes the flow of the code.
The diagram also shows what goes into the actual payload.

Now we need to put together the python script, again using the pwntool module, and deliver the payload.

```
from pwn import process, remote, p32

target = process('./just_do_it')

addrInputStart = 0x28
addrValueToOverwrite = 0x14
paddingRequired = addrInputStart-addrValueToOverwrite
addrOfFlag = 0x0804a080 

print(target.recvuntil(b"password.\n"))

payload = b"A"*paddingRequired + p32(addrOfFlag)

target.sendline(payload)

target.interactive()
```

After you put together the Pyton exploit, just run it and there's the flag!
<img src="https://user-images.githubusercontent.com/101567957/190918668-2c6c4b5c-acff-4ac8-a17b-01297f1573e6.png" width="700">

Wasn't this fun?

