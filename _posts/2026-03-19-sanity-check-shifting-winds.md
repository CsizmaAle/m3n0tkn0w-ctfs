---
title: "UVT CTF – Sanity Check — Shifting Winds"
date: 2026-03-19 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [caesar-cipher, rot, substitution-cipher, dcode, archive-extraction]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Sanity Check — Shifting Winds  
**Category**:  cryptography  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    easy  
**Tags:**  Caesar-cipher, ROT, substitution-cipher, dcode, archive-extraction  
**Date solved:**    19/3/2026 (DD/MM/YYYY)  
**Challenge Description**:  Try to get the first flag extracted from this encrypted file.  
**Files provided**:  Cryptic.rar  

## **Environment and Tools** 
**Operating System**:  Linux  
**Tools/ Software**:  dcode, VirtualBox, Mozilla  
**Programming Languages (if any)**:  -  

## **Recognition Pattern**
- An encrypted file in an archive with no other hints → I start simple and try classical ciphers first
- The ciphertext `VWUJDT{...}` looked almost like the flag format `UVTICS{...}`, and lining up `V → U` is a shift of 1 → Caesar/ROT
- It was one of the first CTFs and a low-effort one, so I figured a classical cipher rather than modern crypto

## **Approach and Methodology**
### **Initial Analysis**
I downloaded the archive and unarchived it. I had a strong feeling it involved encryption.  

### **Steps Taken**
I opened the file inside the archive and found a ciphertext:

```
VWUJDT{X3md0n3}
```

I knew the flag format was `UVTICS{}`, so I assumed it was a Caesar cipher — both because it was one of the first CTFs, and because the shift between the flag template and the ciphertext lined up.  

### **Alternative approach**
I went online and searched for dcode, tried the cipher identifier, and entered the ciphertext. The results weren't helpful, so I went straight for the Caesar cipher.  

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:* ran the ciphertext through the Caesar cipher in dcode and got the plaintext.  

![dcode Caesar cipher decoding the flag](Sanity_Check_1.png)

## **Lessons Learned**
- The habit that saved me time: compare the ciphertext to the flag format before reaching for a tool. I knew flags start with `UVTICS{`, so lining up the letters gave me the shift by hand — faster than any website.
- dcode's cipher identifier didn't get it right on such a short string, so I stopped fighting it and went straight for Caesar.

## **References**
- <https://www.dcode.fr/en>
