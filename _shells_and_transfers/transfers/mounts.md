---
layout: default
title: Mounts
parent: Transfers
nav_order: 1
---

# Mounts

---

## Mount Windows UNC Share

If we have the credentials for a compromised Windows account we can mount a Windows UNC share in the following manner.

```bash
mkdir /tmp/finance
mount -t cifs -o user=almir,password=Corinthians2012,rw,vers=1.0 //172.16.5.10/finance /tmp/finance
ls -l /tmp/finance/
```
