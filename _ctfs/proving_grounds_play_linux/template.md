<!-- ---
layout: default
title: Katana
parent: Proving Grounds Play
nav_order: 2
--- -->

# [Box Name]

---

## Discovered Ports for Enumeration

## Users and Credentials

- _Users_:
- _Credentials_:

# Scanning

---

```bash
sudo nmap -Pn -p- -sC -sV -T4 --open <ip>
```

# Enumeration

---

# Exploitation

---

# Post-Exploitation

---

- [ ] Upgrade to TTY Shell

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

- [ ] sudo -l
- [ ] enumerate users
- [ ] SUID/GUID

```bash
find / -perm -u=s -type f 2>/dev/null
find / -perm -g=s -type f 2>/dev/null
```

- [ ] enumerate process running with root privileges

```bash
ps -aux | grep -i 'root'
```

- [ ] enumerate network services

```bash
netstat -antup (ss -tunlp)
```

- [ ] check for extended capabilities

```bash
getcap -r / 2>/dev/null
```

- [ ] test discovered credentials against all users
- [ ] attempt hydra ssh password crack for discovered users
- [ ] check _/tmp_, _/var/tmp_, _/etc_, and _/dev/shm_
- [ ] Check if /etc/passwd is writeable

```bash
mowree@EvilBoxOne:~$ ls -la /etc/passwd
-rw-rw-rw- 1 root root 1398 ago 16  2021 /etc/passwd

```
