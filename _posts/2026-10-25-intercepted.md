---
title: "RSTCON 2026 – Intercepted"
date: 2026-10-25 08:02:00 +0200
categories: [Events, RSTCON 2026]
tags: [encoding, multi-layer, hex, base64, base32, reverse, cyberchef]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Intercepted  
**Category**:  Crypto  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  encoding, multi-layer, hex, base64, base32, reverse, cyberchef  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  We grabbed a transmission off the wire. It has been through the encoder a few too many times. Peel it back.  
**Files provided**:  transmission.txt  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  Cyberchef  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- The description says "through the encoder a few too many times" / "peel it back" => multiple layers of encoding
- Each output looks like another encoding: = at the end => Base64, ===== at the beginning => reversed Base32

## **Approach and Methodology**
### **Initial Analysis**

I opened the file provided and identified the cipher as hex, so i opened Cyberchef.

```bash
5054303950543039565642505245315452316843527a6379576a525856564a4e55444e5a4e4664614e45784752444a4a57464253546c5261545546594e56704f55444e5a53566861536b383d

```


Recipe: From Hex. The output had a = at the end, so i tried from Base64. 
```bash
PT09PT09VVBPRE1TR1hCRzcyWjRXVVJNUDNZNFdaNExGRDJJWFBSTlRaTUFYNVpOUDNZSVhaSk8=
```

It could have also been Base32 or Base 58.  
The next output started with multiple ===== so i reversed it and applied from Base 32.
```bash
======UPODMSGXBG72Z4WURMP3Y4WZ4LFD2IXPRNTZMAX5ZNP3YIXZJO
```
```bash
OJZXIY3PNZ5XAMZTNRPXI2DFL4ZW4Y3PMRUW4Z27GBXGSMDOPU======
```
## **Solution and Flag Retrieval**
Final Exploit / Command Used:  BAKE! (From Hex → From Base64 → Reverse → From Base32)  
![CyberChef recipe decoding the flag](Intercepted_1.png)

## **Lessons Learned**
- multiple layers can be applied on a message, even reverse.

## **References**
- cyberchef (https://gchq.github.io/CyberChef/)
