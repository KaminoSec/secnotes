---
layout: default
title: Port Knocking
parent: Port Scanning
nav_order: 1
---

# Port Knocking

---

## Test Port Knocking via LFI

Attempt to access the Knock Daemon at _/etc/knockd.conf_ to get the sequence to open a port.

![knockd](../../../assets/images/ctfs/proving_grounds/dc-9/knockd.png)

This gives us the sequence required to unlock the SSH service on the target.

Use the following script to unlock the port via port knocking:

```bash
# Create knock.sh
vim knock.sh

# Add the following bash scripting for loop
for x in 7469 8475 9842; do
    nmap -Pn --host-timeout 201 --max-retries 0 -p $x 192.168.160.209;
done


# Run the knock script
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ ./knock.sh
3Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-09 22:57 EST
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 0 undergoing Host Discovery
Parallel DNS resolution of 1 host. Timing: About 0.00% done
Nmap scan report for 192.168.160.209
Host is up (0.055s latency).

PORT     STATE  SERVICE
7469/tcp closed unknown

Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-09 22:57 EST
Nmap scan report for 192.168.160.209
Host is up (0.054s latency).

PORT     STATE  SERVICE
8475/tcp closed unknown

Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-09 22:57 EST
Nmap scan report for 192.168.160.209
Host is up (0.054s latency).

PORT     STATE  SERVICE
9842/tcp closed unknown

Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds


```

Then we rescan the target and now see that port 22 is open.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY/dc-9]
└─$ nmap -p22 192.168.160.209
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-09 22:57 EST
Nmap scan report for 192.168.160.209
Host is up (0.055s latency).

PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 0.14 seconds
```
