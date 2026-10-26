---
title: "UVT CTF – Mayday Mayday!"
date: 2026-05-19 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [disk-image, file-recovery, steganography, lsb, sleuth-kit, zsteg]
---

## **Challenge Information**
**Title:**  Mayday Mayday!  
**Category**:  forensics  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**  easy  
**Tags:**  disk-image, file-recovery, steganography, LSB, Sleuth-Kit, zsteg  
**Date solved:**  19/05/2026 (DD/MM/YYYY)  
**Challenge Description**:  One of the rocket scientists went crazy and tried to compromise the launch program. All we know is that he deleted the coolant blueprints, which had the reactor key hidden inside them through various methods. Before we got to him he also destroyed the hard disk containing all the data. Fortunately we managed to recover a disk image of the actual hard drive. Do you happen to know a forensics specialist that can help us?  
			Flag format: UVTICS{...}  
**Files provided**:  an image of the hard disk  

## **Environment and Tools** 
**Operating System**:  Kali Linux  
**Tools/ Software**:  VirtualBox, terminal, Mozilla, Sleuth Kit (fls, tsk_recover, icat), file, exiftool, binwalk, strings, xxd, zsteg, 7z  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- A `.img` file → disk forensics
- "deleted the blueprints" → I need to recover deleted files, not just browse the ones that are there
- "hidden inside through various methods" → expect steganography once I've recovered the file
- Put together, the play was recover → identify the target file → check for steg

## **Approach and Methodology**
### **Initial Analysis**
I read the description and downloaded the file. The flag was somewhere inside the `.img`, and the first step was to retrieve the files, including the deleted ones.  

### **Steps Taken**
1. Check the image structure with `file`, `mmls` and `fls`:
```bash
file reactor-blueprints-workstation.img
mmls reactor-blueprints-workstation.img
fls -r reactor-blueprints-workstation.img
```
`fls -r` showed a line with a star next to it: `++++ r/r * 43: coolant-loop-draft.png`. The `*` means the file is deleted. That was my target.

2. Extract all files, including deleted ones:
```bash
mkdir recovered
tsk_recover -e reactor-blueprints-workstation.img recovered/
```
The deleted PNG came back at `recovered/Users/Admin/Desktop/reactor_blueprints/coolant-loop-draft.png`.

3. Investigate the target file:
```bash
file coolant-loop-draft.png
exiftool coolant-loop-draft.png
binwalk coolant-loop-draft.png
```
`binwalk` reported zlib data inside, but that's just normal PNG IDAT data, not an embedded archive. I tried `7z x` and it failed with "Cannot open the file as archive", which confirmed it. Checking with `xxd`, the final bytes corresponded to the IEND chunk, so nothing was appended.

4. Search for LSB steganography with `zsteg`:
```bash
zsteg coolant-loop-draft.png
```
The flag was hidden in the least significant bits.

### **Alternative Approaches (if applicable)**
Could have used Autopsy GUI instead of the Sleuth Kit CLI. Could have used photorec or foremost for file carving if the filesystem metadata was destroyed. Could have used Stegsolve GUI to flip through colour planes manually if zsteg failed. Steghide would NOT have worked because it doesn't support PNG (only JPG/BMP/WAV/AU).  

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:*
```
zsteg coolant-loop-draft.png
```

## **Lessons Learned**
- `tsk_recover -e` was the express path for pulling everything out of the disk image, and the `*` in `fls` output is what flagged the deleted file as my target.
- binwalk false-positived on the PNG because every PNG has a zlib IDAT chunk — I confirmed nothing was actually appended by checking the IEND marker with `xxd` before chasing an extraction.
- Match the steg tool to the format — a PNG here meant `zsteg`, which is where the flag was hiding in the least significant bits.
- Do the cheap checks first: `file` → `exiftool` → `strings` → `binwalk` → `xxd` → steg tools.

## **References**
- <https://www.sleuthkit.org/sleuthkit/docs.php>
- <https://github.com/zed-0xff/zsteg>
