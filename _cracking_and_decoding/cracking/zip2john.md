---
layout: default
title: zip2john
parent: Cracking
nav_order: 1
---

# zip2John

---

Download and unzip looking for password for _creds.txt_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ unzip spammer.zip
Archive:  spammer.zip
[spammer.zip] creds.txt password:

```

Crack with John

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ zip2john spammer.zip > spam.hash
ver 2.0 spammer.zip/creds.txt PKZIP Encr: cmplen=27, decmplen=15, crc=B003611D ts=ADCB cs=b003 type=0

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt spam.hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
myspace4         (spammer.zip/creds.txt)
1g 0:00:00:00 DONE (2023-11-10 10:39) 16.66g/s 307200p/s 307200c/s 307200C/s havana..tanika
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

```

Get creds from _spammer.zip_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ unzip spammer.zip
Archive:  spammer.zip
[spammer.zip] creds.txt password:
 extracting: creds.txt

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ ls
creds.txt  spam.hash  spammer.zip

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ cat creds.txt
mayer:lionheart

```
