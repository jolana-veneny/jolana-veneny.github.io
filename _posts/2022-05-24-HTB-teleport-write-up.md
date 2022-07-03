---
layout: post
tag: WriteUps
author: JLN
---
## HTB's Cyber Apocalypse CTF 2022: Teleport Write-Up
This is a write-up for the Teleport reverse engineering challenge in the HTB Cyber Apocalypse CTF 2022.  
I used Ghidra (and Microsoft Excel) to solve this task.

>You've been sent to a strange planet, inhabited by a species with the natural ability to teleport. If you're able to capture one, you may be able to synthesise lightweight teleportation technology. However, they don't want to be caught, and disappear out of your grasp - can you get the drop on them?

The first step is to upload the binary into Ghidra and have a look at the code in general.

![1_overview](https://user-images.githubusercontent.com/101567957/170101271-dab92b87-ead3-404c-8ad5-03a7679ce100.png)

Straight away, we notice that there is an unusual amount of functions in this binary.
When we click through the functions, we can see that each of them is designed to compare against a specific symbol. In the case of the 00100b2a function that symbol is "t."

![4_functions](https://user-images.githubusercontent.com/101567957/170102073-1d47ccdd-feff-484c-ba82-10b330bc7a8d.png)

That looks very promising. It could of course be unrelated but when we look at the rest of the functions we can see that they all carry letters, numbers, and most importantly brackets that are used in flags. 

![51_Functions](https://user-images.githubusercontent.com/101567957/170102676-08defe32-4e0a-437e-9684-bd2068da9abd.png)

At this point, it is obvious that these functions carry the flag split into individual symbols. The esieast way to put them together (at least for me) is to copy all of these symbols into an excel sheet like this:

![7_symbols](https://user-images.githubusercontent.com/101567957/170103077-bbbed312-f59f-48a8-a17d-4b065007efbe.png)
![8_symbols](https://user-images.githubusercontent.com/101567957/170103087-91fe17f8-cb0d-48e8-b7ef-c4894af95987.png)

It appears this is the entire flag but I was not able to make out the exact order without any other information. So back to Ghidra we go.
Looking at the assembly code, we can see that about half way through the flag functions start being called. And they seem to be called in the correct, flag order, too!

![11_Flag](https://user-images.githubusercontent.com/101567957/170105243-69e90b1b-776f-48ff-a1d7-6698006b273e.png)

![12_Flag2](https://user-images.githubusercontent.com/101567957/170105261-76fc7ab3-881b-430d-a872-0f826a12719d.png)

Now all we have to do is to simply scroll down the assembly code, look up which function is being called one by one, and write down the symbol each of these functions represents until we get to that final "}." 

![13_Flag3](https://user-images.githubusercontent.com/101567957/170105836-8606c413-9dac-47c5-8b12-8a6d7d86ec22.png)![14_Flag4](https://user-images.githubusercontent.com/101567957/170105846-1784a790-4cb4-4457-a643-75fe68273dbe.png)

Clean this all up, and we get the full flag:

![15-Flag_Complete](https://user-images.githubusercontent.com/101567957/170106166-d5b448e8-8704-4466-8d12-6486e87489fb.png)



