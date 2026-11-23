---
title: "WolvCTF 2026 – Maize Poster"
date: 2026-11-21 13:00:00 +0200
categories: [Events, WolvCTF 2026]
tags: [cryptography, steganography, zsteg, lsb, caesar, rot13, dcode]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Maize Poster  
**Category**:  cryptography  
**Platform:**  WolvCTF  
**Difficulty:**  
**Tags:**  cryptography, steganography, zsteg, LSB, caesar, rot13, dcode  
**Date solved:**  21/11/2026 (DD/MM/YYYY)  
**Challenge Description**:  I just got a new poster for my dorm! Though whoever made this put a secret message into it. If you squint really closely at the pixels you might find it.  
**Files provided**:  dis.tar.gzip  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  zsteg, dcode  
**Programming Languages (if any)**:  -  

## **Recognition Pattern**
- "secret message into it / squint at the pixels" => something hidden in the pixels (LSB steganography) => zsteg
- the recovered string is in the flag format but scrambled => a caesar/rot shift on top of the stego layer

## **Approach and Methodology**

### **Steps Taken**
I downloaded the archive and unarchived the file (2 layers). Ran gzip and tar to unarchive it. 

i also ran zsteg on it and came up with a flag but apparently it was wrong:
```bash
zsteg poster.png 
meta Comment        .. text: "Maize poster, 2026."
chunk:0:IHDR        .. file: Adobe Photoshop Color swatch, version 0, 960 colors; 1st RGB space (0), w 0x2e4, x 0x802, y 0, z 0; 2nd space (10061), w 0x27, x 0x4c00, y 0x274d, z 0x27
b1,b,lsb,xy         .. text: "jpgs{e0g13_1a_gu3_oyh3}"
b3,b,lsb,xy         .. file: OpenPGP Secret Key
b4,b,msb,xy         .. text: ["3" repeated 164 times]

```

Then i went to dcode and found out it's encrypted using rot13 or caesar.
```bash
# after rot 13
wctf{r7t80_8n_th0_blu0} - wrong flag

it was caesar (the numbers do not rotate, that's the difference)
```

## **Solution and Flag Retrieval**
Final Exploit / Command Used: caesar decryption in dcode

## **Lessons Learned**
- make sure the flag is somewhat readable :))
- rot13, caesar and rot47 are very similar, the difference is small (what is being rotated): rot13 rotates letters, caesar can include numbers, rot47 rotates punctuation too

## **References**
- zsteg (https://github.com/zed-0xff/zsteg)
- Dcode (https://www.dcode.fr/en)
