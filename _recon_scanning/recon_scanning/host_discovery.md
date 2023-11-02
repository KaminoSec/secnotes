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
