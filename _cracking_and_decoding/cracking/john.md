---
layout: default
title: John
parent: Cracking
nav_order: 4
---

# John

---

## MD5

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/potato]
└─$ nth --file hash.txt

  _   _                           _____ _           _          _   _           _
 | \ | |                         |_   _| |         | |        | | | |         | |
 |  \| | __ _ _ __ ___   ___ ______| | | |__   __ _| |_ ______| |_| | __ _ ___| |__
 | . ` |/ _` | '_ ` _ \ / _ \______| | | '_ \ / _` | __|______|  _  |/ _` / __| '_ \
 | |\  | (_| | | | | | |  __/      | | | | | | (_| | |_       | | | | (_| \__ \ | | |
 \_| \_/\__,_|_| |_| |_|\___|      \_/ |_| |_|\__,_|\__|      \_| |_/\__,_|___/_| |_|

https://twitter.com/bee_sec_san
https://github.com/HashPals/Name-That-Hash


$1$webadmin$3sXBxGUtDGIFAcnNTNhi6/

Most Likely
MD5 Crypt, HC: 500 JtR: md5crypt
Cisco-IOS(MD5), HC: 500 JtR: md5crypt
FreeBSD MD5, HC: 500 JtR: md5crypt


```

Now we can crack with John.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/potato]
└─$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=md5crypt
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 SSE2 4x3])
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dragon           (?)
1g 0:00:00:00 DONE (2023-11-08 22:48) 12.50g/s 1800p/s 1800c/s 1800C/s 123456..sandra
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
