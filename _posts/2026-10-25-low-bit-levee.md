---
title: "RSTCON 2026 – Low Bit Levee"
date: 2026-10-25 08:08:00 +0200
categories: [Events, RSTCON 2026]
tags: [png, steganography, lsb, zsteg]
---

## **Challenge Information**
**Title:**  Low Bit Levee  
**Category**:  steganography (listed as Forensics at RSTCON)  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  png, steganography, LSB, zsteg  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  A levee monitoring station was caught exfiltrating data inside its own status imagery.  
**Files provided**:  levee.png  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  file, strings, binwalk, zsteg  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- The description says the data is hidden "inside its own status imagery" => hidden in the image pixels
- The title "Low Bit" => LSB (least significant bit) steganography
- file, strings and binwalk find nothing => nothing appended, so it's in the pixels => zsteg

## **Approach and Methodology**
The file provided was a png, so i started with file, strings and binwalk to see if anything interesting comes up.

Unfortunately nothing interesting popped up, so i tried zsteg as well.

## **Solution and Flag Retrieval**
Final Exploit / Command Used: zsteg levee.png
```bash
chunk:0:IHDR        .. file: Adobe Photoshop Color swatch, version 0, 256 colors; 1st RGB space (0), w 0xc0, x 0x802, y 0, z 0; 2nd RGB space (0), w 0x1ff, x 0x100, y 0x1ff, z 0xff00
b1,rgb,lsb,xy       .. text: "%[FLAG REDACTED]$"
b2,r,lsb,xy         .. file: raw G3 (Group 3) FAX
b2,g,msb,xy         .. text: "uUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU"
b4,r,lsb,xy         .. text: "33\"3UUDEgvfwh"
b4,g,lsb,xy         .. text: "fgvwvwvgwwvwwgvfvwvwvwfgwgfgwwffwgvwvwvgvgvvwwffvgfvffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff"                                                                                                                             
b4,g,msb,xy         .. text: "fnffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff"
b4,b,lsb,xy         .. text: "#3\"#\"2\"3#3#EUTDEETDEDEfwwwgwwwwww"
b4,b,msb,xy         .. text: ",\"\"\"\"wwwww"
b4,rgb,lsb,xy       .. text: "c6#s'2s6#rV%cV%rG4rG%rf7cv&cg6sv'sgHe"
b4,rgb,msb,xy       .. text: "6ks6gs6g"
b4,bgr,lsb,xy       .. text: "3c&3s7#r&2s&Se&Ru7Bt'Bu6cg&sf7cv&swGehW"
b4,bgr,msb,xy       .. text: "c;vc7vc7"

```

## **Lessons Learned**
- zsteg is a powerful tool for steganography and can identify hidden messages fast

## **References**
- zsteg (https://github.com/zed-0xff/zsteg)
