---
layout: default
title: Brain Fuck
parent: Decoding
nav_order: 1
---

# Brain Fuck

---

Checking the source code we find some text at the bottom of the page:

```bash
!--
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.


-->
```

Use dcode.fr to analyze the text and it determines it is encoded with the Brainfuck cipher.

![decode1](../../../../assets/images/ctfs/proving_grounds/empire-breakout/decode1.png)

Decoding we get a string that looks like a possible password:

![decode2](../../../../assets/images/ctfs/proving_grounds/empire-breakout/decode2.png)
