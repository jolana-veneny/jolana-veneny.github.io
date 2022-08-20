---
layout: post
tag: WriteUps
author: JLN
---
## Crackme WriteUp: Keyg3nMe
This is a write-up of a very easy crackme "keyg3nme."

The original crackme is available [here](https://crackmes.one/crackme/5da31ebc33c5d46f00e2c661) in case you would like to follow along.

It is always a good idea to first figure out what type of file we are working with, in this case I checked it with the `file` command.

<img src="https://user-images.githubusercontent.com/101567957/185750787-818e51b5-aa71-46a4-920a-7393d14dc45e.png" width="600">

Clearly, this is an executable but there is not anything helpful in here.

The next step is to run the executable to see how it behaves.

<img src="https://user-images.githubusercontent.com/101567957/185750952-568816c0-0cfa-4e5e-8f79-04bd248f476d.png" width="600">

As you can see, the executable asks for a password and if you fail to provide the correct one, it just says "nope." Which is rude but at least we now know how this executable behaves.

This still does not tell us anything about what the password might be so we now move on to Ghidra.
After we open a new project and analyze the file, this is what we see:

![Keyg3nMe_Ghidra](https://user-images.githubusercontent.com/101567957/185752028-9d0b5f5e-133e-4af0-be91-9665bcc8789d.png)

Straight away, let's jump into the `main` function.

<img src="https://user-images.githubusercontent.com/101567957/185752075-99fdd45c-40ab-4041-8092-6a78bd9b1e17.png" width="400">

Here, we can already see the code's internal logic. The function takes the user-provided `input_pass` variable, uses it as a parameter for the `validate_key` function and if it comes back as equal to 1, the function prints `"Good job mate, now go keygen me."` If it is not equal to 1, it prints `nope.`
A quick note, the return value of the `validate_key` function here is a boolean value and so equal to 1 means that the `validate_key` function needs to be true.

So let's have a look at the `validate_key` function. 

![Keyg3nme_validate_c](https://user-images.githubusercontent.com/101567957/185754474-20d204c6-d29c-4203-95f4-f398c84894c0.png)

As we can see the `validate_key` function simply checks whether `param_1 % 1223 == 0`, that is the password is divisible by 1223. If it is, it returns true (1) and if it is not it returns false (0).

Logically then, the password is 1223. But from the way the validation logic is written it can also be 0 (because 0 mod 1223 = 0) or 2446 (2446 mod 1223 = 0) or any other mulitple of 1223.

This gets us the return value of true (1) in `validate_key` and the main function confirms we have the right password.

<img src="https://user-images.githubusercontent.com/101567957/185755304-1f92a95c-8e59-4aa7-a9fb-248cab6a2ee8.png" width="600">

<img src="https://user-images.githubusercontent.com/101567957/185755307-da138b34-dd00-41c0-8f80-1411dbfefd04.png" width="600">

