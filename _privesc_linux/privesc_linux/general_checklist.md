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

## System and Network Information

```bash
hostname

# kernel version
uname -a

# OS verion
cat /etc/issue

# Running processes
ps auxw
ps -aux | grep -i 'root'

# Network routes
route -n

# DNS server
cat /etc/resolv.conf

# Arp cache
arp -a

# Network connections
netstat -antup
ss -tunlp
```

## User Information

```bash
# Current user permissions
find / -user username

# UID and GID Information for all users
for user in $(cat /etc/passwd |cut -f1 -d":"); do id $user; done

# Last logged on User
last -a

# Root accounts (how many UID 0 accounts are on the system)
cat /etc/passwd |cut -f1,3,4 -d":" |grep "0:0" |cut -f1 -d":" |awk '{print $1}'

# Do any service accounts have shells defined? Can we login as those accounts?
cat /etc/passwd

# Do we have access to any other users' home directories?
ls -als /home/*

```

## Privileged Access/Cleartext Credentials

```bash
# Check current user elevated privileges
sudo -l

# SUID binaries vulnerable to privilege escalation
find / -perm -4000 2>/dev/null
find / -perm -u=s -type f 2>/dev/null

# Check for extended capabilities
getcap -r / 2>/dev/null

# Can we read configuration files that contain sensitive information, passwords, etc.?
grep "password" /etc/*.conf 2>/dev/null
grep -r "password" /etc/*.conf 2>/dev/null

# Can we read the shadow file? Can we crack any hashes?
cat /etc/shadow

# Can we list or read the contents of the /root directory?
ls -als /root

# Can we read other users history files?
find /* -name *.*history* -print 2>/dev/null

# Grep apache access.log for "user" and "pass" strings
cat /var/log/apache/access.log |grep -E "^user|^pass"

# Check for aliases (e.g. root)
compgen -a

```

## Services

```bash
# Which services are configured on the system and what ports are open?
netstat -auntp
ss -tunlp

# Are service configuration files readable or modifiable by current user?
find /etc/init.d/ ! -uid 0 -type f 2>/dev/null |xargs ls -la

# Do configuration files contain any useful information?
cat /etc/mysql/my.cnf

# Can we start/stop services?
service service_name start/stop
```

## Jobs/Tasks

```bash
# What tasks/jobs is the system configured to run and how often?
cat /etc/crontab
ls -als /etc/cron.*

# Are there any custom jobs or tasks configured as root that are world-writeable?
find /etc/cront* -type f -perm -o+w -exec ls -l {} \;

```

## Installed Software Version Information

```bash
# What software packages are installed on the system?
dpkg -l


```

## Misc.

- [ ] test discovered credentials against all users
- [ ] attempt hydra ssh password crack for discovered users
- [ ] check _/opt_, _/tmp_, _/var/tmp_, _/etc_, and _/dev/shm_
- [ ] Check if /etc/passwd is writeable
