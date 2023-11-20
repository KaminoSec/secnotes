---
layout: default
title: Port Scanning
has_children: true
nav_order: 2
---

# Port Scanning

---

## Nmap

### Service and version detection plus script scan

```bash
sudo nmap -Pn -p- -sC -sV -T4 -vv --open 192.168.235.136
```

- -Pn: no ping
- -sC: script scan using the default set of Nmap scripts; equivalent to --script=default
- -sV: service version detection
- --open: only display open ports (not filtered or closed)
- -T4: aggressive scanning speed
- -vv: verbose output

### SYN (Stealth) scan

```bash
nmap -Pn -p- -sS -T4 -vv --open 10.12.1.36
```

- -sS: SYN scan; never completes the TCP handshake by sending RST instead of ACK

### UDP scan

```bash
nmap -Pn -sU -vv -T4 --open 10.12.1.36
```

- -sU: UDP scan
