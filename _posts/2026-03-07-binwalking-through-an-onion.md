---
title: "UVT CTF – Binwalking through an Onion"
date: 2026-03-07 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [binwalk, hidden-files, archive, embedded-data, rot13, caesar-cipher, hexed.it, 7z, file-carving]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Binwalking through an Onion  
**Category**:  steganography  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    easy  
**Tags:**  binwalk, hidden-files, archive, embedded-data, ROT13, Caesar-cipher, hexed.it, 7z, file-carving  
**Date solved:**    07/03/2026 (DD/MM/YYYY)  
**Challenge Description**:  Onions have so many layers, what could they hide inside?  
**Files provided**:  a .png file of an onion  

## **Environment and Tools** 
**Operating System**:  Linux  
**Tools/ Software**:  hexed.it, dcode, 7z  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- "Layers" / "onion" in the description → something hidden inside, probably an embedded file or nested archive
- The image looked normal but the challenge hinted otherwise → data appended or embedded past the image's real end
- Scrolling to the bottom of the hex and seeing `flag.txt` → there's an archive embedded inside the file
- Extracted text that's gibberish but only letters → a simple substitution cipher, so I try ROT13 first

## **Approach and Methodology**
### **Initial Analysis**
There seemed to be nothing wrong with the picture, but the "layers" in the description made me think about something hidden within the file's metadata.  

### **Steps Taken**
I uploaded the file to hexed.it and scrolled all the way to the bottom, where I found something interesting: some text containing `flag.txt`. That made me think about archives, so I decided to try to unarchive `BigOnion.png`.  

![flag.txt visible at the bottom of the hex in hexed.it](Binwalking_Onion_1.png)

I first gave it execute rights, then unarchived it with 7z:
```bash
7z x BigOnion.png
```

A new file popped up in the folder, `flag.txt`, but it contained a string that looked like a cipher. So I went to dcode and used the cipher identifier.  

![dcode cipher identifier suggesting Caesar/ROT13](Binwalking_Onion_2.png)

There was a high chance it was a Caesar cipher with a 13-position rotation, so I went to check that out.  

### **Alternative Approaches (if applicable)**
The first time I solved this CTF, I found `flag.txt` by changing the extension from `.png` to `.txt` and scrolling to the end of the file. The second time that didn't work, so I had to use hexed.it.  

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:* in dcode, I entered the ciphertext in ROT13 and got the plaintext flag.  

## **Lessons Learned**
- Files can hide other files appended after their real data ends — the image viewer ignores the extra bytes, but `7z` finds and extracts them.
- `7z x` pulled the embedded archive out regardless of the `.png` extension — it just looks for archive signatures inside, which is exactly what the "binwalking" in the name is hinting at.
- The extracted `flag.txt` was ROT13 — worth trying that first when text comes out as letters-only gibberish, since it's the go-to "easy final layer" in these.
- I gave the file execute rights before extracting, but I didn't actually need to — `7z x` only reads the file; `chmod +x` is only for running scripts or binaries.

## **References**
- <https://www.dcode.fr/en>
- <https://hexed.it/>
