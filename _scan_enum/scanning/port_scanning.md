---
layout: default
title: Port Scanning
nav_order: 2
---

# Port Scanning

---

## Nmap - Initial Scanning

### Service and version detection plus script scan

```bash
sudo nmap -Pn -p- -sC -sV -T4 -vv --open 192.168.235.136
```

- -Pn: no ping
- -sC: script scan using the default set of Nmap scripts; equivalent to --script=default
- -sV: service version detection
- --open: only display open ports (not filtered or closed)
- -T4: aggressive scanning speed
- -vv: verbose output

### SYN (Stealth) scan

```bash
nmap -Pn -p- -sS -T4 -vv --open 10.12.1.36
```

- -sS: SYN scan; never completes the TCP handshake by sending RST instead of ACK

### UDP scan

```bash
nmap -Pn -sU -vv -T4 --open 10.12.1.36
```

- -sU: UDP scan

## Nmap Scripting Engine

Update script database

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ sudo nmap --script-updatedb
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-29 21:41 EST
NSE: Updating rule database.
NSE: Script Database updated successfully.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.58 seconds

```

SMB-OS Discovery

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ sudo nmap --script smb-os-discovery -p 445 192.168.1.6

```

SMB-Enum-Shares

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ sudo nmap --script smb-enum-shares -p 445 192.168.1.6

```
