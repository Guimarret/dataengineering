---
title: "Logic gates"
date: 2024-01-16T15:04:23-03:00
draft: false
---

Using the transistor base in here: [transistor]( {{< ref "/posts/transistor.md" >}}). 

I'm gonna talk about logic gates and some basic possibilities like adders, subtractotons, multiplier and divisor. The main idea in these basic posts are trace the way to assembly instructions and so on.

These logic gates are the lowest level we can reach in the hardware processing, obviously these instructions alone don't create anything really meaninfull but the correct combination can create computer and processing machines of any type (except quantum computer but we're not gonna talk about them today)  

So logic gates are simply a transistor sequence that creates properties (AND, OR, NAND, NOR, XOR and XNOR)

- The X in the name means exclusively something, like in OR gate that usually would accept 1|1 but with X it'll only except the 1|0 (doesn't matter the factor order) 
- The N means the opposite of the original logic gate output.
- In the bottom of the image there are some "grounds" that a recommend to look in <a href="https://www.build-electronic-circuits.com/what-is-ground/" target="_blank"> this </a> explication before continuing 
- The +6V is the place where eletricity comes from and the out is where they come out if the logic gate sustain the correct input (the input is just the electrical presence in A or B that'll activate the transitor)

Transistor level approach:
- AND (The first input and the second simultaneously):
    ![](/img/and.png)
- OR (The first input, the second or both):
    ![](/img/or.png)
- NAND (The first input, the second or none of them):
    ![](/img/nand.png)
- NOR (Both of them off):
    ![](/img/nor.png)

Truth tables:
![](/img/logic_gates.png)

I didn't put XOR and XNOR transistor level image because they use the other basic gates and i think it'll confuse more than help right now.

Now you can be thinking something like:

-"Why would a even have to know this"

And i tell you that this is literally the base of any normal computer (quantum computer is the only different that i think rn but we're not gonna talk about then today)

Now i'm gonna take some real approach for something more likely understandable, the *adder*:

## Adder

- If you need some basic idea help in binary mathematics this
<a href="https://byjus.com/maths/binary-operation/" target="_blank"> site </a> can help.

The adders uses this logic gates setup (i'm gonna start using this images with de minimal representation of the logic gates because it would become a bothersome to use so much space with logic gates at transistor level):
![](/img/adder.png)

Basically the adder receive 3 inputs, the binary of the first number, the second and the "carry in" (cin), but if it's the first comparison the cin is ignored, because the carry in is the carried number from the last operation, aka the COUT that appears in the end of the circuit, so for a better visualization this is the truth table:
![](/img/adder_truth_table.png)

So, if we have 2 numbers, 1001 and 1000 the adder circuit will compare the far right number from both this case it'll be 1 and 0, resulting in S = 1 and Cout = 0, creating the first bit of the output that is S (1).
With this we reach the conclusion that the adder will be runned for every bit we have in our sum operation, this situation we have 2 numbers of 4 bits so the adder is gonna be runned 4 times to get the full output/result.

If you want to see some real implementation of this adder there is this <a href="https://www.youtube.com/watch?v=X31B1pVow1o" target="_blank"> incredible video </a>

## Subtractor

### In progress...




Transistor images refference:
http://hyperphysics.phy-astr.gsu.edu/hbase/Electronic/trangate.html#c1
