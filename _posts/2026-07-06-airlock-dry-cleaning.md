---
title: "UVT CTF – Airlock Dry Cleaning"
date: 2026-07-06 08:02:00 +0200
categories: [CTF, UVT CTF]
tags: [forensics, disk-image, file-recovery, deleted-files, sleuth-kit, tsk_recover]
---

## **Challenge Information**
**Title:**  Airlock Dry Cleaning  
**Category**:  forensics  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**  easy  
**Tags:**  forensics, disk-image, file-recovery, deleted-files, Sleuth-Kit, tsk_recover  
**Date solved:**  06/07/2026 (DD/MM/YYYY)  
**Challenge Description**:  We recovered a damaged hard disk and managed to convert it to a disk image. Try to extract anything relevant from it so we can continue with the case.  
			Flag format: UVTICS{...}  
**Files provided**:  usb-image.img  

## **Environment and Tools** 
**Operating System**:  Kali Linux  
**Tools/ Software**:  Sleuth Kit, binwalk  
**Programming Languages (if any)**:  -  

## **Recognition Pattern**
- A `.img` disk image with "extract anything relevant" → disk forensics, and "recovered a damaged disk" is a strong hint to go after deleted files, not just the ones still listed

## **Approach and Methodology**
### **Initial Analysis**
A `.img` was provided, so I went straight for the obvious forensics move: see what files are inside it, including anything deleted.

### **Steps Taken**
1. `file usb-image.img` to confirm what I was dealing with.
2. `binwalk usb-image.img` — nothing interesting came up.
3. This felt just like the **Mayday Mayday** challenge, so I reached for the Sleuth Kit's express recovery:

```bash
tsk_recover -e usb-image.img recovered/
```

4. That pulled out 3 files, so I looked through them. The flag was in `_ECRET.TXT` — which is `SECRET.TXT` with its first character mangled by whatever damaged the disk.

### **Alternative Approaches (if applicable)**
I found a second way completely by accident: just running `cat usb-image.img` prints the flag straight into the terminal, since it sits in plaintext somewhere on the image.

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:*

```bash
cat recovered/_ECRET.TXT
```

## **Lessons Learned**
- Sometimes there's an easy way out — the whole thing came down to one recovery command.
- Try `cat` on the image itself too; the flag was readable in the raw `.img` without recovering anything.
- `tsk_recover -e` is my go-to for pulling deleted files out of a disk image, same as it was in Mayday Mayday.

## **References**
none used
