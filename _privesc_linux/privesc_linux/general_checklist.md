---
layout: default
title: General Checklist
nav_order: 1
---

# General Checklist

---

### UPgrade TTY Shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
ctrl+z
stty size
stty raw -echo
fg
stty rows <first_value>
stty cols <second_value>
export TERM=xterm-256color
```

### SUID executables

```bash
find / -perm -4000 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
```

### Enumerate processes

```bash
ps -aux | grep -i 'root'
```

### Enumerate network services

```bash
netstat -antup
ss -tunlp
```

### Check for extended capabilities

```bash
getcap -r / 2>/dev/null
```

### Misc.

- [ ] test discovered credentials against all users
- [ ] attempt hydra ssh password crack for discovered users
- [ ] check _/opt_, _/tmp_, _/var/tmp_, _/etc_, and _/dev/shm_
- [ ] Check if /etc/passwd is writeable
- [ ] Check for aliases (e.g. root)

```bash
compgen -a
```
