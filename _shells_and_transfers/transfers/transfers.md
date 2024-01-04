---
layout: default
title: Transfers
nav_order: 1
has_children: true
---

This is the Transfers section

## Netcat File Transfer

```bash
# Target Machine (receiving file)
nc -l -p 12334 > LinEnum.sh

# Attack Machine (sending file)
nc -w 3 <target_ip> 1234 < LinEnum.sh

# Set the executable bit
chmod +x LinEnum.sh
```
