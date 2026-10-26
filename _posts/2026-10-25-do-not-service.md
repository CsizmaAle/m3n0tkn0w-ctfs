---
title: "RSTCON 2026 – Do Not Service"
date: 2026-10-25 08:13:00 +0200
categories: [Events, RSTCON 2026]
tags: [pcap, wireshark, tshark, dns, dns-exfiltration, base64, python]
---

## **Challenge Information**
**Title:**  Do Not Service  
**Category**:  networking (listed as Forensics at RSTCON)  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  pcap, wireshark, tshark, DNS, DNS-exfiltration, base64, python  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  The new IT guy said not to use this computer, I used it anyways.  
**Files provided**:  odd_transmissions.pcap  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  wireshark, tshark  
**Programming Languages (if any)**:  bash, python  

## **Recognition Pattern**
- A huge pcap and searching for rstcon{ finds nothing => the flag is encoded somewhere
- The title "Do Not Service" => DNS
- DNS queries with weird subdomains that look like Base64 => data exfiltrated through DNS names

## **Approach and Methodology**
Since the file provided was a pcap, i decided to open it in wireshark. The file provided was extremely long, so it was out of question to look through it manually.

I tried searching for the string rstcon{ , but it came out empty. It would have been too simple.

 The name of the challenge was interesting, it pointed to DNS, so i applied that filter, together with only queries.
 ```
 # initial search
 dns.flags.response == 0 
 
 #advanced search; there were a lot of .local names and broadcasting between macbooks, so i filtered it out and applied the ip of the interesting found names
dns.flags.response == 0 && !(dns.qry.name contains "local") && ip.dst==3.143.112.103
 ```
  
  I ended up finding some interesting names that looked like Base64 ciphertext. 

To extract those names, i used a tshark command (recommended by claude.ai), and a grep to clean the names up:
```bash
tshark -r "odd_transmissions.pcap" -Y "ip.dst==3.143.112.103 && dns.flags.response==0" -T fields -e dns.qry.name > names.txt

grep -oP '(?<=www\.)[^.]+(?=\.)' names.txt > labels.txt
```

```bash
AA1NYXR0aGV3IFlvdW5n
ABNDaHJpc3RvcGhlciBNaXJhbmRh
ABBTdGVwaGFuaWUgT2JyaWVu
ABFNcnMuIE5hbmN5IE1pbGxlcg==
AAlKbyBOZXd0b24=
AAxIZWF0aGVyIFNpbXM=
AAxKb3NodWEgSm9uZXM=
AA5aYWNoYXJ5IFphbW9yYQ==
AAtEYXZpZCBKb25lcw==
AA1Sb2JlcnQgTmd1eWVu
ABFEci4gU2FyYWggV2Vic3Rlcg==
AA9Kb2hubnkgUGhpbGxpcHM=
ABBTdGVwaGFuaWUgR2FyY2lh
ABBLaW1iZXJseSBTYWxhemFy
AA1Sb2JlcnQgTWlsbGVy
AA1KdWRpdGggV2F0c29u
AA5UYW1teSBDaGFuZGxlcg==
AAxEcmV3IFN1bW1lcnM=
AAxTYXJhIE1jY2xhaW4=
ABFDaHJpc3RvcGhlciBMZXdpcw==
ABBCaWxseSBNZW5kZXogRERT
AAxEYXJpdXMgQ2xhcms=
AAlKYW1lcyBMZWU=
AB1yc3Rjb257ZDBtYWluX25AbTNfRVhQYW5zaTBufQ==
AAxUaW5hIEJvbmlsbGE=
AA1DYXJsb3MgTWVkaW5h
AA1TaGFyaSBIaWdnaW5z
AA5EYW5pZWxsZSBGcm9zdA==
AA1BZGFtIEdyaWZmaXRo
AA5NaWNoZWxsZSBCbGFjaw==

```

To read them, i created a python script that decrypts them and prints them out.

```python3
#!/usr/bin/env python3

import base64

input="labels.txt"

with open (input) as f:
	for line in f:
		line=line.strip()
		decoded = base64.b64decode(line)
		text = decoded.decode("utf-8")
		print(text)
```

To run the solver i changed the permissions.
## **Solution and Flag Retrieval**
Final Exploit / Command Used: ./solver.py
```bash
Matthew Young
Christopher Miranda
Stephanie Obrien
Mrs. Nancy Miller
        Jo Newton

Heather Sims

Joshua Jones
Zachary Zamora

David Jones
Robert Nguyen
Dr. Sarah Webster
Johnny Phillips
Stephanie Garcia
Kimberly Salazar
Robert Miller
Judith Watson
Tammy Chandler

Drew Summers

Sara Mcclain
Christopher Lewis
Billy Mendez DDS

Darius Clark
        James Lee
[FLAG REDACTED]

Tina Bonilla
Carlos Medina
Shari Higgins
Danielle Frost
Adam Griffith
Michelle Black

```

## **Lessons Learned**
- names can hide information
- DNS queries can be used to exfiltrate data, hidden in the subdomain names

## **References**
- claude.ai (https://claude.ai)
- Wireshark / tshark (https://www.wireshark.org)
