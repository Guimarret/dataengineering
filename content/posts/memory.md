---
title: "Memory"
date: 2024-02-06T23:10:24-03:00
draft: True
---

So, this is a extensive and sometimes boring matter but also really important..

I`m gonna start with latches and flip-flops and hoperfully end in ram/sram

## Latch/Flip-flops

For a fresh start and just for contextualization, the difference between both is that the Latch don't have a clock (initially can be any energy pulse, but today it's used quartz crystal oscillation) but i'll explain further later.

I'm gonna make this post progressive, starting from the base latch to Flip-flop, D type, JK type and finally T type

### S-R

Uses two inputs, Set and Reset.
![](/img/memory/sr-flip-flop-logic-diagram.jpg)

There are a couple of details here that we have to think about, so the idea in the SR type is to make the output always the opposite, and looking at a bigger picture the circuit to store a 32 bits is needed 32 of these and get the output of Q for example to get the actual data (don't really matter if is the Q or -Q here because the system is symmetrical so is just arbitrarily decided at the first time and later on is just replicated). If you need to erase the values you can just pass the reset signal for every latch.

(We can't let them both be 0 because it would return an invalid output and also if both are 1 it also means it's an error because at logic level it can't be possible)

### Flip-Flop version

Have 3 inputs:

![](/img/memory/sr-flip-flop-logic-circuit.jpg)

Just for a glance we can tell that the difference is the AND gates in the left and the Clk (clock), the logic addition have the purpose to accept signal only when the clock send the signal, so we can get a tempo of the execution creating a more solid utilization for real world situations.

### D

In the D type, we change the S input name to D with a reverse logic gate in and the R to E, creating a new output of truth table that it's impossible to have invalid states as before (1|1 and 0|0 input)

![](/img/memory/internal-logic-d-latch.webp)

And the E can be though as the clock for this version.

### JK

