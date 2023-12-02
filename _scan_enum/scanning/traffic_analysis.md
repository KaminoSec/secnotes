---
layout: default
title: Packet Traffic Analysis
nav_order: 7
---

# Packet Traffic Analysis

---

## HTTP Traffic

From the Arpspoof traffic captured in _Sniffing and MITM_ section we have intercepted traffic between 172.16.5.5 and the default gateway.

This includes HTTP packets captured that went to 10.10.10.10.

We can use the following filter to display HTTP traffic from the source address 172.16.5.5

```bash
http and ip.addr == 172.16.5.5
```

![http](../../../assets/images/ctfs/proving_grounds/sniffing/http.png)

We can filter on all HTTP GET requests with the following filter

```bash
http.request.method == "GET"
```

![get](../../../assets/images/ctfs/proving_grounds/sniffing/get.png)

We can filter on all HTTP POST requests with the following filter

```bash
http.request.method == "POST"
```

![post](../../../assets/images/ctfs/proving_grounds/sniffing/post.png)

If we follow the HTTP Stream we can see the clear text credentials

![creds](../../../assets/images/ctfs/proving_grounds/sniffing/creds.png)

We can export files by selecting _File_ then Export Objects > HTTP

![export](../../../assets/images/ctfs/proving_grounds/sniffing/export.png)

## SMB Files

We can filter out files trasmitted by _SMB_ using the following filter:

```bash
smb.file
```

![smb](../../../assets/images/ctfs/proving_grounds/sniffing/smb.png)

We can export SMB file objects by selecting Export Objects > SMB

![smbfile](../../../assets/images/ctfs/proving_grounds/sniffing/smbfile.png)
