---
layout: post
tag: WriteUps
author: JLN
---
## Crackme WriteUp: PleaseCrackMe
Another Crackme in C called "PleaseCrackMe"

This time I chose a CrackMe which, even though it is still fairly simple, requires the ability to understand C. The CrackMe is called "PleaseCrackMe" and is available to download at [the CrackMes.one website.](https://crackmes.one/crackme/612e85d833c5d41acedffa4f)

Before diving into the actual analysis, let's have a look at how the executable behaves:

<img src="https://user-images.githubusercontent.com/101567957/185801576-f57d1bd0-a3b9-418b-877b-edb5d6a05f1e.png" height="180">

As we can see, the user is asked to enter a username, then to pick a number between 1 and 9, and to enter a password. As expected, typing random answers will get you the "Wrong answer" response.

Now let's load the executable directly into Ghidra. When we look at the `main` function in the decompiler, we can see straight away that it contains some interesting code that has a bunch of if statements and directly deals with the password.

![main_function_original](https://user-images.githubusercontent.com/101567957/185801573-ffe6252d-ffe0-4902-9f24-c0c57f25e475.png)

The easiest way to tackle this is simply to identify the variables, one by one to get a better sense of what is going on. 
This is where some basic knowledge of C is key. 

We can start with renaming the input variables which store the user answers. Based on what answer the scanf function captures, we can identify each variable. The variable `local_78` becomes `username`, variable `local_80` is the `number_chosen`, and  `local_38` is the password the user types in, `answer_password`. 

Next up is `local_7c = local_7c + 1;` this structure is very common and it is used as a loop counter in C, so we can rename `local_7c` to `loop-counter`.

At this point, we can see that right before the if condition which determnines whether we get the password or get denied is this little string comparison function: `iVar1 = strcmp(local_58,answer_password);`. Seeing as the value of `iVar1` determines whether the password is correct or not, and that the comparison is run against the password the user provided, we can assume that `local_58` is the actual password. The last step here is then to rename `local_58` to `password`. 

This is what we have now:

![main_function_c](https://user-images.githubusercontent.com/101567957/185802718-bc0ae732-ea1a-467f-a0bf-90446c6e2378.png)

The code is now very clear. The first section deals with input and input validation, the middle part is a while loop, and the last part is password validation. Focusing on the while loop, we can see that this is the part where the password is actually being created, meaning the password is not static. Look at this code:

`password[loop-counter] = (char)number_chosen + username[loop-counter];`

What this tells us is that the password consists of each character from the username to which we add the number we picked, all of that converted back into a character. So when the user name is "a" and we picked number 1, the password is "b". For username "hello" and number 1, the password is "ifmmp".
This is nothing more than a simple Caesar/substitution cipher. 

<img src="https://user-images.githubusercontent.com/101567957/185803905-522673a9-b498-4b15-ba14-c0438a62bfca.png" height="180">

And there you have it, this was a fun little CrackMe with a simple crypto twist!
