---
layout: default
title: Medusa
parent: Cracking
nav_order: 7
---

# Medusa

---

[Medusa Github Repo](https://github.com/jmk-foofus/medusa)

[Medusa User Manual](http://foofus.net/goons/jmk/medusa/medusa.html)

Here are some common commands

![commands](../../../assets/images/ctfs/proving_grounds/medusa/commands.png)

To list all the available modules and to list information about a specific module use the following commands.

```bash
medusa -M

medusa -M telnet -q
```

## Telnet

```bash
medusa -h 192.168.102.149 -M telnet -U username.txt -P password.txt
```
