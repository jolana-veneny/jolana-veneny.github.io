---
layout: post
tag: WriteUps
author: JLN
---

## WriteUp: Assembly1 PicoCTF
A writeup of a short Assembly exercise

This picoCTF challenge is about testing your assembly reading skills and the original is at [this website.](https://play.picoctf.org/practice/challenge/111?category=3&page=1&search=) 

These are the instructions:

> For what argument does this program print `win` with variables 81, 0 and 3? File: chall_1.S Flag format: picoCTF{XXXXXXXX} -> (hex, lowercase, no 0x, and 32 bits. ex. 5614267 would be picoCTF{0055aabb})

When you open the chall_1.S file in a text editor, this is what you will see: 

<p float="left">
  <img src="https://user-images.githubusercontent.com/101567957/193464534-507ef980-7eba-466c-b14e-e36ec79dd455.png" width="40%" />
  <img src="https://user-images.githubusercontent.com/101567957/193464536-623ebf78-c059-40f4-a80b-e089ef998350.png" width="50%" align="top" /> 
</p>



Now, this can seem a bit overwhelming. So let's start by looking at the basic logic in the script's `main` function.
The key part in the main function is this: 
```
	cmp	w0, 0			# if the cmp (compare) function between w0 and 0
	bne	.L4			# is NOT true (bne - branch if not equal) it will take us to L4
	adrp	x0, .LC0		# if it IS true it will take us to .LC0
	add	x0, x0, :lo12:.LC0
```

We can see that .LC0 is where we want to go:
```
.LC0:
	.string	"You win!"
	.align	3
```
The other option .L4 takes us to
``` 
.L4:
	adrp	x0, .LC1
	add	x0, x0, :lo12:.LC1
	bl	puts
```
and by extension to
```
.LC1:
	.string	"You Lose :("
	.text
	.align	2
	.global	main
	.type	main, %function
```


This means that if we want to win, we need the w0 variable to be 0. We also know that the w0 variable is stored at `[x29, 28]`, meaning at the address of stack + 28. 
At this point, we know how this script is organized logically but we still need to get to the numbers.

The answer is in the `func` function and this requires going through the code line by line like I did in my notes:
<img src="https://user-images.githubusercontent.com/101567957/193464880-76525178-e694-4a88-81cd-a7aad8ef3236.jpg" width="90%" />


The first couple of lines are just variables being initiated, variable `[stack + 12]` to user input, variable `[stack + 16]` to 81, variable `[stack + 20]` to 0, and variable `[stack + 24]` to 3. 

Then, we see the `lsl` instruction which stands for logical shift left. [This website](http://www-mdp.eng.cam.ac.uk/web/library/enginfo/mdp_micro/lecture4/lecture4-3-2.html) has a great explanation of what it is and how it works. In this case the `lsl` instruction has these parameters: w0, w1, w0. As is usually the case in assembly, the first parameter, w0, is where the result is stored. The second, w1, is the variable we are changing, and the third, w0 again, is the number we shift by. 
In this case, w1 is the `[stack +16]` variable which stores 81, and we are shifting by the value of the `[stack + 20]` which is 0. So yes, we are shifting 81 by 0, meaning that this entire paragraph has been much ado about nothing. Moving on.

The next instruction is `sdiv` which stands for Signed Divide, and the instruction syntax is similar to lsl. The first parameter is the result destination, the second one is the dividend (the number we are dividing), the third is the divisor (the number we divide by). Again, we are dividing w1, in this case `[stack + 28]`, by w0, `[stack + 24]`. Currently, these variables hold the values 81 and 3, making the result at `[stack + 28]` 27.

The last instruction in this section is `sub`, meaning subtract, the w1 `[stack + 28]` - the w0 `[stack + 12]`. This means we get 27 - the value of user input. This result then gets stored in `[stack + 28]`.

Now, we can go back to the `main` function. We know that if we want to get to the `win` branch, our w0 at `[stack + 28]` needs to equal zero. Knowing that the `func` function results in 27, it is clear that to get the flag, we need to input 27 as the initial arguement. 27 is 1b in hex and following the instructions, we submit picoCTF{0000001b} as the flag.

And that's the right answer!
