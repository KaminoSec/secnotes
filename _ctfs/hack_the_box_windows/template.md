<!-- ---
layout: default
title: Katana
parent: Proving Grounds Practice - Linux
nav_order: 2
--- -->

# Scanning

---

```bash
sudo nmap -Pn -p- -sC -sV -T4 --open <ip>
```

# Enumeration

---

# Exploitation

---

# Post-Exploitation

---

[PayloadsAllTheThings Guide](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)

[Absolomb Windows Privesc Guide](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)

[Sushant 747's Guide](https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html)

- systeminfo

```bash
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"
```

- wmic qfe,logicaldisk

```bash
wmic qfe Caption,Description,HotFixID, InstalledOn

wmic logicaldisk get caption,description,providername
```

- whoami /priv
- whoami /groups
- net user
- net localgroup; net localgroup administrators
- [ ]

- [ ]
- [ ]
- [ ]
- [ ]
- [ ]
