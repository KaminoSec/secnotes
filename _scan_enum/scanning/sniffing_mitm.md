---
layout: default
title: Sniffing and MITM
nav_order: 6
---

# Sniffing and MITM

---

## Cain and Abel

Click _Start/Stop Sniffer_

![start](../../../assets/images/ctfs/proving_grounds/sniffing/start.png)

Discovered hosts appear in the _Sniffer_ tab

![hosts](../../../assets/images/ctfs/proving_grounds/sniffing/hosts.png)

### Arp Poisoning

From the _APR_ tab select the top table area and then the _plus_ icon to start an ARP Poisoning attack.

![plus](../../../assets/images/ctfs/proving_grounds/sniffing/plus.png)

Select the target on the left (172.16.2.5) and the default gateway (172.16.5.1) on the right.

![target1](../../../assets/images/ctfs/proving_grounds/sniffing/target1.png)

Repeat this process until all the hosts for the ARP Poisoning attack have been populated.

![table](../../../assets/images/ctfs/proving_grounds/sniffing/table.png)

Click the start button to begin the ARP attack

![start2](../../../assets/images/ctfs/proving_grounds/sniffing/start2.png)

![attack](../../../assets/images/ctfs/proving_grounds/sniffing/attack.png)

Stop the attack by clicking the _Sniffer_ and _ARP_ buttons

Click the _passwords_ tab to see the captured passwords

![password](../../../assets/images/ctfs/proving_grounds/sniffing/password.png)

Click on the _smb_ filter and right-click > _send to cracker_ to crack captured hashes

![crack](../../../assets/images/ctfs/proving_grounds/sniffing/crack.png)

Click the _Cracker_ tab and then right-click > _dictionary attack_

![dictionary](../../../assets/images/ctfs/proving_grounds/sniffing/dictionary.png)

Right-click on the _Dictionary Table_ and select _Add to List_

![list](../../../assets/images/ctfs/proving_grounds/sniffing/list.png)

Select the wordlist and then the _start_ button; after awhile the cracked password should appear.

![cracked](../../../assets/images/ctfs/proving_grounds/sniffing/cracked.png)

Select the _Network_ tab and click _Quick List_ and then _Add to Quick List_

![quick](../../../assets/images/ctfs/proving_grounds/sniffing/quick.png)

Connect to the server

![connect](../../../assets/images/ctfs/proving_grounds/sniffing/connect.png)

Right-click on _Services_ and install _Abel_

![abel](../../../assets/images/ctfs/proving_grounds/sniffing/abel.png)

Double click on the server and _Abel_ should appear where we can run commands from the console.

![whoami](../../../assets/images/ctfs/proving_grounds/sniffing/whoami.png)
