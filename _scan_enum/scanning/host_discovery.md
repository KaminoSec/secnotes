---
layout: default
title: Host Discovery
nav_order: 1
---

# Host Discovery

---

## Net Discover

```bash
sudo netdiscover -i eth0 -r 10.0.2.0/24
```

![netdiscover](../../../assets/images/netdiscover.png)

## Nmap Host Discover

```bash
sudo nmap -sn 10.0.2.0/24
```

## arp-scan

```bash
sudo arp-scan -I eth0 -g 10.0.2.0/24
```

![arpscan](../../../assets/images/arpscan.png)

## fping

```bash
fping -I eth0 -g 10.0.2.0/24 -a 2>/dev/null
```

![fping](../../../assets/images/fping.png)

## hping3

Single port Syn scan

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PRACTICE/rubydome]
└─$ sudo hping3 -S -p 80 -c 3 192.168.173.22
HPING 192.168.173.22 (tun0 192.168.173.22): S set, 40 headers + 0 data bytes
len=40 ip=192.168.173.22 ttl=61 DF id=0 sport=80 flags=RA seq=0 win=0 rtt=63.8 ms
len=40 ip=192.168.173.22 ttl=61 DF id=0 sport=80 flags=RA seq=1 win=0 rtt=62.7 ms
len=40 ip=192.168.173.22 ttl=61 DF id=0 sport=80 flags=RA seq=2 win=0 rtt=64.1 ms

--- 192.168.173.22 hping statistic ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 62.7/63.5/64.1 ms

```
