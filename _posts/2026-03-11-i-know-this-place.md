---
title: "UVT CTF – I Know This Place!"
date: 2026-03-11 08:01:00 +0200
categories: [CTF, UVT CTF]
tags: [steganography, corrupted-header, hex-editing, exif, osint, image-repair]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  I Know This Place!  
**Category**:  steganography  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    easy  
**Tags:**  steganography, corrupted-header, hex-editing, EXIF, OSINT, image-repair  
**Date solved:**    11/03/2026 (DD/MM/YYYY)  
**Challenge Description**:  Someone sent me a photo, seems to be corrupted. They also hinted that I might need to find the place where the photo was taken in order to get the whole flag...  
**Files provided**:  an image that won't open  

## **Environment and Tools** 
**Operating System**:  Linux (VirtualBox + Ubuntu)  
**Tools/ Software**:  Firefox, hexed.it, Google Images  
**Programming Languages (if any)**:  -  

## **Recognition Pattern**
- Image that won't open → probably a corrupted or modified header (magic bytes)
- "corrupted" in the description → check the file header against the known signature
- "find the place" + a photo → an OSINT/geolocation step (reverse image search, visible landmarks) on top of the repair
- Worth checking the end of the file too — flag fragments are often appended after the real data

## **Approach and Methodology**
### **Initial Analysis**
Opening the file in Firefox gave me an error message, so not much to see there. I saved it locally and tried again, but it still failed. At this point I figured the metadata was the issue, but couldn't tell exactly what.  

### **Steps Taken**
The first step after downloading was to change the extension from `.jpg` to `.txt` in order to view the metadata. The first thing I noticed was the header: `MAGI\00JFIF\00\`. I knew it was the wrong one, but not what the correct one was. I kept scrolling to the end of the document (as I'd done in the "Binwalking through an Onion" CTF), and sure enough there was part of the flag: `UVTICS{1_L0V3_V1SIT1NG_(CITYNAME)}`.  

After that, I went online for a hex editor, found <https://hexed.it/>, and uploaded the image.  

![the corrupted header in hexed.it](I_Know_This_Place_1.png)

I looked up the header for JPG files (`FF D8 FF`), fixed it in hexed.it, and deleted the end of the file (the part with the flag fragments). I opened the image in the site's live preview window and got this:  

![the repaired image previewed](I_Know_This_Place_2.png)

### **Alternative Approaches**
- Could try editing hex directly in a text editor, but this often corrupts binary data — use a proper hex editor.
- `xxd` or `hexedit` in the terminal would also work instead of hexed.it.
- `exiftool` could reveal GPS coordinates if the EXIF data was intact.

Another method might have been changing the header and deleting the end of the file directly in notepad, but when I converted it back to `.jpg` the file still failed to open.  

## **Solution and Flag Retrieval**
I took a screenshot of the repaired picture and uploaded it to Google Images, which gave me the city: **Paris**. I then realized the city was written in the bottom-right corner of the photo.  

*Final Exploit / Command Used:* repaired the JPG header (`FF D8 FF`) in hexed.it, then reverse-image-searched the photo to recover the city name and complete the flag.  

## **Lessons Learned**
- When an image won't open, I check the magic bytes first — mine had a mangled header (`MAGI\00JFIF...`), and fixing it to the real JPEG signature `FF D8 FF` in hexed.it repaired it.
- Check the end of the file too — that's where the flag fragment `UVTICS{..._(CITYNAME)}` was appended, the same trick I hit in the Binwalking Onion challenge.
- Use a proper hex editor for binary files, not a text editor — when I tried the header fix in notepad and saved back to `.jpg`, the file still wouldn't open.
- This one was two steps in one: repair the file, then OSINT the city. The `(CITYNAME)` placeholder in the flag was the tell that there was more work than just the technical fix, so I reverse-image-searched to get Paris.

## **References**
- <https://hexed.it/>
- <https://www.file-recovery.com/jpg-signature-format.htm>
- <https://images.google.com/>
