---
layout: default
title: 21 | FTP
parent: Service Enumeration
nav_order: 1
---

# 21 | FTP

---

## Anonymous Logon

Simply provide "anonymous" parameter for Name prompt

```bash
└─$ ftp 192.168.206.107
Connected to 192.168.206.107.
220 ProFTPD 1.3.5e Server (Debian) [::ffff:192.168.206.107]
Name (192.168.206.107:vagrant): anonymous
331 Anonymous login ok, send your complete email address as your password
Password:
230-Welcome, archive user anonymous@192.168.45.217 !
230-
230-The local time is: Wed Nov 01 02:17:49 2023
230-
230-This is an experimental FTP server.  If you have any unusual problems,
230-please report them via e-mail to <root@funbox2>.
230-
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

## Download files

Use the 'get' parameter to download files from the target machine to your local attack machine.

```bash
ftp> get welcome.msg
local: welcome.msg remote: welcome.msg
229 Entering Extended Passive Mode (|||27652|)
150 Opening BINARY mode data connection for welcome.msg (170 bytes)
100% |***********************************************************|   170        3.68 MiB/s    00:00 ETA
226 Transfer complete
170 bytes received in 00:00 (2.87 KiB/s)

```

## Upload files

Use the 'put' parameter to upload files from your local attack machine to the target machine.

```bash
ftp> put rev.php
```
