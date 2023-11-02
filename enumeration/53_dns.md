---
layout: default
title: 53 | DNS
nav_order: 2
---

# 53 | DNS

---

## Nslookup

```bash
# quick lookup
> nslookup website.com

# full lookup (all record types)
> nslookup -type=any website.com
```

## Host

Return IP addresses associated with a domain name or conduct a zone transfer

```bash
> host google.com

# reverse lookup
> host 10.10.10.10

# more indepth
host -l friendzone.red 10.10.10.123 > zonetransfer.txt
host -l friendzoneportal.red 10.10.10.123 >> zonetransfer.txt

# zone transfer; retrieve name servers for a domain
> host -t axfr -l google.com ns1.google.com

```

## Dig

```bash
# retrieve name servers
> dig -t any google.com

# test for zone transfer
> dig axfr google.com ns1.google.com
```

## Fierce

Locate non-contiguous IP space and hostnames using DNS

```bash
> fierce -dns google.com
```

Here are the things that Fierce does:

- locate the name servers for a given domain
- attempt a zone transfer on every name server
- checks for a wildcard DNS record
- brute forces subdomains using an internal wordlist

## DNSenum

```bash
> dnsenum google.com
```

Dnsenum does the following checks on a domain:

- host addresses
- name servers
- mx servers
- zone transfers

## Sublist3r

```bash
# install
> apt update && apt -y install sublist3r

# run sublist3r
> sublist3r -d google.com -b -t 100
```

- -d = domain name to enumerate subdomains
- -b = brute-forcing with Subbrute
- -t = number of threads

## LDAP Enumeration

```bash
# initial search
> ldapsearch -h 10.10.10.161 -x -s base namingcontexts

# after getting the domain name, get the sAMAccountName records
> ldapsearch -h 10.10.10.161 -x -b "DC=htb, DC=local" '(objectClass=Person)' sAMAccountName
```

- -h = host
- -x = simple authentication
- -s = scope
