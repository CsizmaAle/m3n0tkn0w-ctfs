---
title: "WolvCTF 2026 – The Very Meta CTF"
date: 2026-11-23 09:03:00 +0200
solved: 2026-11-21 08:03:00 +0200
categories: [Events, WolvCTF 2026]
tags: [forensics, metadata, exiftool, exif, user-comment, hex, cyberchef]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  The Very Meta CTF  
**Category**:  forensics  
**Platform:**  WolvCTF  
**Difficulty:**  
**Tags:**  forensics, metadata, exiftool, EXIF, User-Comment, hex, cyberchef  
**Date solved:**  21/11/2026 (DD/MM/YYYY)  
**Challenge Description**:  (forgot to copy but something regarding files from a pirate's computer)  
**Files provided**:  dis.tar.gzip  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  exiftool, Cyberchef  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- the title and description point at metadata => the flag is hidden in the files' metadata (EXIF), not the pixels
- each image carries a piece in its User Comment, and the pieces are hex => collect them in order and decode

## **Approach and Methodology**
### **Initial Analysis**
The title and description hint at metadata, so i went straight for exiftool on the provided files.

### **Steps Taken**
Ran gzip and tar to unarchive the two layers. And obtained a folder with multiple files. Since the title of the challenge points at metadata i decided to skip ahead there.   
Ran a for loop to see all the metadata at once:
```bash
for f in pictures/*; do echo "\n== $f=="; exiftool "$f" ;done 

== pictures/eight.png==
ExifTool Version Number         : 13.55
File Name                       : eight.png
Directory                       : pictures
File Size                       : 2.1 kB
File Modification Date/Time     : 0000:00:00 00:00:00
File Access Date/Time           : 2026:11:21 16:12:41+02:00
File Inode Change Date/Time     : 2026:11:21 16:08:10+02:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : 7763
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Image Size                      : 400x400
Megapixels                      : 0.160

== pictures/five.png==
ExifTool Version Number         : 13.55
File Name                       : five.png
Directory                       : pictures
File Size                       : 6.6 kB
File Modification Date/Time     : 0000:00:00 00:00:00
File Access Date/Time           : 2026:11:21 16:12:41+02:00
File Inode Change Date/Time     : 2026:11:21 16:08:10+02:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : 7466
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Image Size                      : 400x400
Megapixels                      : 0.160

== pictures/four.png==
ExifTool Version Number         : 13.55
File Name                       : four.png
Directory                       : pictures
File Size                       : 6.7 kB
File Modification Date/Time     : 0000:00:00 00:00:00
File Access Date/Time           : 2026:11:21 16:12:41+02:00
File Inode Change Date/Time     : 2026:11:21 16:08:10+02:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : 7b74
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Image Size                      : 400x400
Megapixels                      : 0.160

== pictures/nine.png==
ExifTool Version Number         : 13.55
File Name                       : nine.png
Directory                       : pictures
File Size                       : 7.7 kB
File Modification Date/Time     : 0000:00:00 00:00:00
File Access Date/Time           : 2026:11:21 16:12:41+02:00
File Inode Change Date/Time     : 2026:11:21 16:08:10+02:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : 6831
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Image Size                      : 400x400
Megapixels                      : 0.160

== pictures/one.png==
ExifTool Version Number         : 13.55
File Name                       : one.png
Directory                       : pictures
File Size                       : 7.8 kB
File Modification Date/Time     : 0000:00:00 00:00:00
File Access Date/Time           : 2026:11:21 16:12:41+02:00
File Inode Change Date/Time     : 2026:11:21 16:08:10+02:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : 735f
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Image Size                      : 400x400
Megapixels                      : 0.160

== pictures/seven.png==
ExifTool Version Number         : 13.55
File Name                       : seven.png
Directory                       : pictures
File Size                       : 6.1 kB
File Modification Date/Time     : 0000:00:00 00:00:00
File Access Date/Time           : 2026:11:21 16:12:41+02:00
File Inode Change Date/Time     : 2026:11:21 16:08:10+02:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : 3173
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Image Size                      : 400x400
Megapixels                      : 0.160

== pictures/six.png==
ExifTool Version Number         : 13.55
File Name                       : six.png
Directory                       : pictures
File Size                       : 4.3 kB
File Modification Date/Time     : 0000:00:00 00:00:00
File Access Date/Time           : 2026:11:21 16:12:41+02:00
File Inode Change Date/Time     : 2026:11:21 16:08:10+02:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : 5f34
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Image Size                      : 400x400
Megapixels                      : 0.160

== pictures/ten.png==
ExifTool Version Number         : 13.55
File Name                       : ten.png
Directory                       : pictures
File Size                       : 5.2 kB
File Modification Date/Time     : 0000:00:00 00:00:00
File Access Date/Time           : 2026:11:21 16:12:41+02:00
File Inode Change Date/Time     : 2026:11:21 16:08:10+02:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : 5f66
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Image Size                      : 400x400
Megapixels                      : 0.160

== pictures/three.png==
ExifTool Version Number         : 13.55
File Name                       : three.png
Directory                       : pictures
File Size                       : 7.0 kB
File Modification Date/Time     : 0000:00:00 00:00:00
File Access Date/Time           : 2026:11:21 16:12:41+02:00
File Inode Change Date/Time     : 2026:11:21 16:08:10+02:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : 6c34
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Image Size                      : 400x400
Megapixels                      : 0.160

== pictures/two.png==
ExifTool Version Number         : 13.55
File Name                       : two.png
Directory                       : pictures
File Size                       : 6.3 kB
File Modification Date/Time     : 0000:00:00 00:00:00
File Access Date/Time           : 2026:11:21 16:12:41+02:00
File Inode Change Date/Time     : 2026:11:21 16:08:10+02:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 400
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Exif Version                    : 0232
Components Configuration        : Y, Cb, Cr, -
User Comment                    : 675f4945306a3332657d
Flashpix Version                : 0100
Color Space                     : Uncalibrated
Image Size                      : 400x400
Megapixels                      : 0.160

```

Here i found something interesting. There were comments spread across the files. i decided to collect them, in this specific weird order, and put them in CyberChef. Since they were all 0-9 and a-f it was most likely hex.
## **Solution and Flag Retrieval**
Final Exploit / Command Used: bake!
![assembling the hex User Comments into the flag in CyberChef](Very_Meta_CTF_1.png)

## **Lessons Learned**
 - trust the title of the challenge and the description
 - metadata (EXIF User Comment) is an easy place to hide data across several files
## **References**
- exiftool (https://exiftool.org)
- Cyberchef (https://gchq.github.io/CyberChef/)
