---
layout: default
title: 161 | SNMP
parent: Service Enumeration
nav_order: 4
---

# 161 | SNMP

---

## Initial Scan

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ sudo nmap -sU -p 161 10.4.22.142
Starting Nmap 7.92 ( https://nmap.org ) at 2023-11-30 10:46 IST
Nmap scan report for 10.4.22.142
Host is up (0.0091s latency).

PORT    STATE SERVICE
161/udp open  snmp
```

## Brute Force Community Strings

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ sudo nmap -sU -p 161 --script=snmp-brute 10.4.22.142
Starting Nmap 7.92 ( https://nmap.org ) at 2023-11-30 10:48 IST
Nmap scan report for 10.4.22.142
Host is up (0.0089s latency).

PORT    STATE SERVICE
161/udp open  snmp
| snmp-brute:
|   public - Valid credentials
|   private - Valid credentials
|_  secret - Valid credentials

Nmap done: 1 IP address (1 host up) scanned in 1.72 seconds

```

## SNMPWalk

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ snmpwalk -v 1 -c public 10.4.22.142

```

## Nmap SNMP Scripts

We can get useful information from the Nmap SNMP scripts, such as user names.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ sudo nmap -sU -p 161 --script snmp-* 10.4.22.142 > snmp_output
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ cat snmp_output

| snmp-win32-users:
|   Administrator
|   DefaultAccount
|   Guest
|   WDAGUtilityAccount
|_  admin
```
