---
layout: default
title: 80 | HTTP
nav_order: 3
---

# 80 | HTTP

---

## Initial Checklist

- [ ] /robots.txt
- [ ] /.DS_STORE
- [ ] /.svn
- [ ] /sitemap.xml
- [ ] /crossdomain.xml
- [ ] /clientaccesspolicy.xml
- [ ] Check comments in source code
- [ ] Review certificates of HTTPS sites

## Nmap Vuln Scan

```bash
nmap --script vuln -p80,443 -oA vuln-scan 10.10.10.79
```

## http

Returns the http header

```bash
http 10.10.242.194
```

## Whatweb

```bash
whatweb 10.10.242.194
```

![whatweb](../../../assets/images/whatweb.png)

## Gobuster

```bash
# Seclists
gobuster dir -u http://10.10.10.187/utility-scripts -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php,txt,html -t 80

# Dirbuster lowercase medium
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirbubster/directory-list-lowercase-2.3-medium.txt -x aspx, php, txt, conf - t 80

# CGI-Bin for Shellshock
gobuster dir -u http://192.168.202.83:8088/cgi-bin/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -k -t 80 -x sh,cgi
```

Common Flags:

- -k: skip SSL certificate verification
- -t: number of threads (default = 10)
- -w: path to wordlist
- -f: append '/' to each request
- -x: file extension to search for
- -U: username for basic auth
- -P: password for basic auth

## wfuzz

### Fuzzing files

```bash
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt --hc 404 "http://192.168.202.83/FUZZ"
```

### Fuzzing directories

```bash
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt --hc 404 "http://192.168.202.83/FUZZ"
```

### Fuzz Keyword for LFI Fuzzing

```bash
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt --hh 0 "http://192.168.202.80/console/file.php?FUZZ=../../../../../../../etc/passwd"
```

- --hh: hush hush; ignore 0 character byte responses (double dash)

## Hydra Brute Force

Brute force HTTP Login page if there is not a CSRF token

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt -f 209.97.132.64 -s 31009 http-post-form "/admin_login.php:user=^USER^&pass=^PASS^:F=<form name='login'"
```

## Nikto

```bash
nikto -o 'output.txt' -p 80 -h 10.10.10.161
```

## Cewl

Wordlist creator based on website keywords

```bash
cewl http://10.10.10.161 -w output.txt
```

## Common Web Config Files

### PHPInfo

- phpinfo.php
- Shows lots of useful information, including the DOCUMENT_ROOT path

![phpinfo](../../../assets/images/phpinfo.png)

### Wordpress PHP Config

- /var/www/html
- wp-config.php
- Should contain 'db-user' password

### PHPmyAdmin

- /etc/phpmyadmin
- config-db.php

if this is installed with WordPress, the 'wordpress' user might have access (wp-config.php)
{: .warn }
