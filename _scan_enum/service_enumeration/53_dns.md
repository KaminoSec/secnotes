---
layout: default
title: 53 | DNS
parent: Service Enumeration
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

# DNS Server Discovery

If we have a list of live hosts and check which one has TCP/53 open.

- TCP/53 is used for _zone transfers_
- UDP/53 is used for _DNS queries_

## Discover DNS Server

Therefore, we want to find the live host that has TCP/53 open.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ sudo nmap -sC -sV -p 53 --open 172.16.5.1,5,6,10

Nmap scan report for 172.16.5.10

PORT   STATE SERVICE VERSION
53/tcp open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.16.1-Ubuntu
MAC Address: 08:00:27:4A:45:F3 (Oracle VirtualBox virtual NIC)

```

## Determine Domain Name

Now that we now 172.16.5.10 is the DNS server we want to identify the network Domain Name.

We can do that with either _nslookup_ or _dig_

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ nslookup
> server 172.16.5.10
Default server: 172.16.5.10
Address: 172.16.5.10#53
> 172.16.5.5
5.5.16.172.in-addr.arpa name = wkst-techsupport.sportsfoo.com.16.172.in-addr.arpa.

```

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ dig @172.16.5.10 -x 172.16.5.5 +nocookie

; <<>> DiG 9.16.15-Debian <<>> @172.16.5.10 -x 172.16.5.5 +nocookie
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36416
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;5.5.16.172.in-addr.arpa.       IN      PTR

;; ANSWER SECTION:
5.5.16.172.in-addr.arpa. 86400  IN      PTR     wkst-techsupport.sportsfoo.com.16.172.in-addr.arpa.

;; Query time: 0 msec
;; SERVER: 172.16.5.10#53(172.16.5.10)
;; WHEN: Fri Dec 01 13:53:52 EST 2023
;; MSG SIZE  rcvd: 116
```

## Execute Zone Transfer

Using _dig_ to conduct a Zone Transfer we will find hosts in a different network.

- 10.10.10.x

We didn't previously have this information from our _ARP_ scan due to our ARP broadcast requests being limited to the same broadcast domain. Therefore, the 10.10.10.x network was unreachable.

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ dig @172.16.5.10 sportsfoo.com -t AXFR +nocookie

; <<>> DiG 9.16.15-Debian <<>> @172.16.5.10 sportsfoo.com -t AXFR +nocookie
; (1 server found)
;; global options: +cmd
sportsfoo.com.          86400   IN      SOA     ine-labserver.sportsfoo.com. hostmaster.sportsfoo.com. 2011071001 3600 1800 604800 86400
sportsfoo.com.          86400   IN      NS      ine-labserver.sportsfoo.com.
ftp.sportsfoo.com.      86400   IN      A       10.10.10.6
ine-labserver.sportsfoo.com. 86400 IN   A       172.16.5.10
intranet.sportsfoo.com. 86400   IN      A       10.10.10.10
wkst-finance.sportsfoo.com. 86400 IN    A       172.16.5.6
wkst-techsupport.sportsfoo.com. 86400 IN A      172.16.5.5
sportsfoo.com.          86400   IN      SOA     ine-labserver.sportsfoo.com. hostmaster.sportsfoo.com. 2011071001 3600 1800 604800 86400
;; Query time: 48 msec
;; SERVER: 172.16.5.10#53(172.16.5.10)
;; WHEN: Fri Dec 01 13:55:26 EST 2023
;; XFR size: 8 records (messages 1, bytes 276)

```

## Identify the Default Gateway for 172.16.5.0/24 Network

Using traceroute

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ traceroute 10.10.10.10 -m 5
traceroute to 10.10.10.10 (10.10.10.10), 5 hops max, 60 byte packets
 1  172.16.5.1 (172.16.5.1)  0.453 ms  0.369 ms  0.332 ms
 2  10.10.10.10 (10.10.10.10)  0.620 ms  0.819 ms  0.807 ms

# If there is an issue sending ICMP packets include the -T for TCP packets
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ traceroute 10.10.10.10 -m 5 -T
traceroute to 10.10.10.10 (10.10.10.10), 5 hops max, 60 byte packets
 1  172.16.5.1 (172.16.5.1)  0.455 ms  0.359 ms  0.327 ms
 2  10.10.10.10 (10.10.10.10)  0.704 ms  0.673 ms  0.636 ms

```

We can check the route table as well to determine the default gateway

```bash
┌──(vagrant㉿kali)-[~/Documents/PG/PLAY]
└─$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.0.1     0.0.0.0         UG    0      0        0 adlab0
10.10.10.0      172.16.5.1      255.255.255.0   UG    0      0        0 eth1
172.16.5.0      0.0.0.0         255.255.255.0   U     0      0        0 eth1
192.168.0.0     0.0.0.0         255.255.255.0   U     0      0        0 adlab0

```

## Draw a Network Diagram

![network](../../../assets/images/ctfs/proving_grounds/sniffing/network.png)
