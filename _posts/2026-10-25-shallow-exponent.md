---
title: "RSTCON 2026 – Shallow Exponent"
date: 2026-10-25 08:03:00 +0200
categories: [CTF, RSTCON 2026]
tags: [rsa, small-e, e=3, cube-root, gmpy2, factordb, python]
---

## **Challenge Information**
**Title:**  Shallow Exponent  
**Category**:  Crypto  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  RSA, small-e, e=3, cube-root, gmpy2, factordb, python  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  We intercepted the RSA handshake off an RST Hydro telemetry link. Somebody rolled their own RSA.  
**Files provided**:  shallow-exponent.txt  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  factordb, gmpy2, pycryptodome  
**Programming Languages (if any)**:  bash, python  

## **Recognition Pattern**
- RSA challenge with n, e, c given, and e is very small (e = 3)
- The title says "Shallow Exponent" and n can't be factorized (factordb has nothing)
- c is a lot shorter than n => m^e never got bigger than n, so the mod didn't do anything

## **Approach and Methodology**
### **Initial Analysis**
This challenge was about RSA and decrypting a message.

The file provided contained:
```bash
RST Hydro telemetry link, RSA handshake intercept
n = 1798833870873869284849918845620860538643880165022986742114642813661312853315304820782679618029532208957490095534130436612379372009999529099477747067768691526264177407415393442030762679500180205561694238201499359974897638676308345557663717049176534190057523522294384352445526791295392361693622465896436707
e = 3
c = 2327530519918477406206611427021204828261125512460673959207517503179638080356935424394665845069203565812949721743509027944983080555335276025128649854048470750403183372527635554979736435865413380593355053898690963586326808006858109846817125

```

One thing that stood out was the fact that when i tried factordb to get the two factors needed for the decryption, it showed no factors available. So there was that.

One thing that i took as a clue was the fact that it said "Shallow exponent", so maybe it was not needed to factorize it. What i found interesting was the value "e". It usually is a big value (65537), not something as small as 3. 

For this one, Claude.ai helped again. The logic is to find the e-th root of c, to replace the pow(c,d,n) that is usually used for decryption (it works because m^3 is smaller than n, so c is just m^3). To do this, i used gmpy2.iroot().

```python3
#!/usr/bin/env python3
from Cryptodome.Util.number import long_to_bytes
import gmpy2


# RST Hydro telemetry link, RSA handshake intercept

n = 1798833870873869284849918845620860538643880165022986742114642813661312853315304820782679618029532208957490095534130436612379372009999529099477747067768691526264177407415393442030762679500180205561694238201499359974897638676308345557663717049176534190057523522294384352445526791295392361693622465896436707

e = 3

c = 2327530519918477406206611427021204828261125512460673959207517503179638080356935424394665845069203565812949721743509027944983080555335276025128649854048470750403183372527635554979736435865413380593355053898690963586326808006858109846817125

  
m, exact = gmpy2.iroot(c, e) # e-th root of c, replaces pow(c, d, n)

print(exact) # True means it worked

print(long_to_bytes(m))
```

To run the script i had to change the permissions: chmod +x ./solver.py.
## **Solution and Flag Retrieval**
Final Exploit / Command Used:  ./solver.py
```bash
True
b'[FLAG REDACTED]'

```

## **Lessons Learned**
- if the e is a vulnerable value (a small number like 3) it can be exploited and the message can be decrypted.
- gmpy2.iroot(c,e) can be used in cases such as shallow exponent.

## **References**
- claude.ai (https://claude.ai)
- factordb (http://factordb.com)
