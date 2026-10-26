---
title: "RSTCON 2026 – Layered Catch"
date: 2026-10-25 08:07:00 +0200
categories: [Events, RSTCON 2026]
tags: [png, appended-zip, binwalk, zsteg, strings, foremost, metadata]
---

## **Challenge Information**
**Title:**  Layered Catch  
**Category**:  steganography (listed as Forensics at RSTCON)  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  png, appended-zip, binwalk, zsteg, strings, foremost, metadata  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  Someone sent us a harmless looking image. It opens fine, shows nothing interesting, and yet it weighs more than a 64 by 64 teal square has any right to.  
**Files provided**:  catch.png  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  file, binwalk, zsteg, strings, foremost  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- The description says the image is bigger than it should be => something is hidden/appended inside it
- binwalk shows a Zip archive after the PNG data (extra data after IEND)
- The PNG has text chunks (Comment, Passphrase) => check the metadata too

## **Approach and Methodology**
### **Initial Analysis**
A png file is provided with the disclaimer that it seems bigger than it should be. So i checked it with file and binwalk.

```bash
┌──(alecybersec㉿bad)-[~/…/2026_events/RSTCON/Forensics/LayeredCatch]
└─$ file catch.png
catch.png: PNG image data, 64 x 64, 8-bit/color RGB, non-interlaced
                                                                                                                                                                                                              
┌──(alecybersec㉿bad)-[~/…/2026_events/RSTCON/Forensics/LayeredCatch]
└─$ binwalk catch.png 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 64 x 64, 8-bit/color RGB, non-interlaced
153           0x99            Zlib compressed data, best compression
248           0xF8            Zip archive data, encrypted at least v1.0 to extract, compressed size: 45, uncompressed size: 33, name: flag.txt
453           0x1C5           End of Zip archive, footer length: 22


```

Since binwalk has sometimes issues extracting files and archives, i used zsteg to try and get the answer without extracting the archive.

```bash
[?] 227 bytes of extra data after image end (IEND), offset = 0xf8
extradata:0         .. file: Zip archive data, made by v3.0 UNIX, extract using at least v1.0, last modified Oct 22 2026 15:14:34, uncompressed size 33, method=store
    00000000: 50 4b 03 04 0a 00 09 00  00 00 d1 79 56 5d 6a 30  |PK.........yV]j0|
    00000010: 23 a1 2d 00 00 00 21 00  00 00 08 00 1c 00 66 6c  |#.-...!.......fl|
    00000020: 61 67 2e 74 78 74 55 54  09 00 03 9a 60 da 6a 3b  |ag.txtUT....`.j;|
    00000030: 72 97 6a 75 78 0b 00 01  04 e8 03 00 00 04 e8 03  |r.jux...........|
    00000040: 00 00 fd 6d 8a 23 58 21  18 f0 84 2d f2 a0 75 7a  |...m.#X!...-..uz|
    00000050: 36 3f ae f0 cf 59 68 88  07 a7 cb c6 b4 c7 47 c1  |6?...Yh.......G.|
    00000060: 1c 43 c6 62 71 21 32 22  76 a3 a5 36 34 fa 4d 50  |.C.bq!2"v..64.MP|
    00000070: 4b 07 08 6a 30 23 a1 2d  00 00 00 21 00 00 00 50  |K..j0#.-...!...P|
    00000080: 4b 01 02 1e 03 0a 00 09  00 00 00 d1 79 56 5d 6a  |K...........yV]j|
    00000090: 30 23 a1 2d 00 00 00 21  00 00 00 08 00 18 00 00  |0#.-...!........|
    000000a0: 00 00 00 01 00 00 00 b4  81 00 00 00 00 66 6c 61  |.............fla|
    000000b0: 67 2e 74 78 74 55 54 05  00 03 9a 60 da 6a 75 78  |g.txtUT....`.jux|
    000000c0: 0b 00 01 04 e8 03 00 00  04 e8 03 00 00 50 4b 05  |.............PK.|
    000000d0: 06 00 00 00 00 01 00 01  00 4e 00 00 00 7f 00 00  |.........N......|
    000000e0: 00 00 00                                          |...             |
meta Comment        .. text: "decoy: the catch is layered, look past the picture"
meta Passphrase     .. text: "[FLAG PART]"

```

One other way to get the middle part of the flag is to use:
```bash
foremost -i catch.png -o output/
```

And look for the flag file.

When i tried strings on the image it also showed the existence of the flag file:
```bash
IHDR
:tEXtComment
decoy: the catch is layered, look past the picture
tEXtPassphrase
[FLAG PART]
OIDATx
IEND
yV]j0#
flag.txtUT
uz6?
bq!2"v
yV]j0#
flag.txtUT
```

Another way is to use the Aperi'Solve website: https://www.aperisolve.com/2fb80f6a94578a0a7fc8bc9e489e6935 
## **Solution and Flag Retrieval**
Final Exploit / Command Used: zsteg catch.png  
 Putting the flag together rstcon{ + flag.txt +}

## **Lessons Learned**
- sometimes there is a faster way than extracting the archive and unzipping

## **References**
- zsteg (https://github.com/zed-0xff/zsteg)
- binwalk (https://github.com/ReFirmLabs/binwalk)
- Aperi'Solve(https://www.aperisolve.com/)
