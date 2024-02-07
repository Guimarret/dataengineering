---
title: "Memory"
date: 2024-02-06T23:10:24-03:00
draft: true
---

---
title: "Memory"
date: 2024-02-05T15:07:31-08:00
draft: false
---

So, this is a extensive and sometimes boring matter but also really important..

I`m gonna start with latches and flip-flops and hoperfully end in ram/sram

## Latch/Flip-flops

For a fresh start and just for contextualization, the difference between both is that the Latch don't have a clock (initially can be any energy pulse, but today it's used quartz crystal oscillation) but i'll explain further later.

### Latch Types

### S-R

Uses two inputs, Set and Reset.
![](/img/memory/sr-flip-flop-logic-diagram.jpg)

There are a couple of details here that we have to think about:

We can't let them both be 0 because it would return an invalid output

### D

Single input and transfer the Data input when the