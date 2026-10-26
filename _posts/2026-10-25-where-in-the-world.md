---
title: "RSTCON 2026 – Where in the World"
date: 2026-10-25 08:11:00 +0200
categories: [CTF, RSTCON 2026]
tags: [osint, geolocation, exif, metadata, exiftool, gps]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Where in the World  
**Category**:  OSINT  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  OSINT, geolocation, EXIF, metadata, exiftool, GPS  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  A contact sent us this "vacation photo." The picture itself is nothing, but the photographer was not careful about what their camera wrote into it. Figure out exactly where this was taken.  
500m Tolerance  
**Files provided**:  vacation.jpg  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  exiftool, Mozilla  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- The description says the camera "wrote into" the photo => EXIF metadata, possibly with GPS
- "500m Tolerance" => the answer is a set of coordinates

## **Approach and Methodology**
Since the description hints to the possibility of metadata not being erased, i decided to look into it, hoping to find the GPS location. 
```bash
exiftool vacation.jpg

ExifTool Version Number         : 13.55
File Name                       : vacation.jpg
Directory                       : .
File Size                       : 23 kB
File Modification Date/Time     : 2026:10:25 10:07:18+02:00
File Access Date/Time           : 2026:10:25 10:07:41+02:00
File Inode Change Date/Time     : 2026:10:25 10:07:27+02:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Exif Byte Order                 : Big-endian (Motorola, MM)
Make                            : Canon
Camera Model Name               : Canon EOS R7
Software                        : field-capture 2.1
Modify Date                     : 2026:05:14 18:42:11
Date/Time Original              : 2026:05:14 18:42:11
User Comment                    : caught the sunset with one foot in each state
GPS Version ID                  : 2.3.0.0
GPS Latitude Ref                : North
GPS Longitude Ref               : West
GPS Altitude Ref                : Above Sea Level
Image Width                     : 640
Image Height                    : 480
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 640x480
Megapixels                      : 0.307
GPS Altitude                    : 335 m Above Sea Level
GPS Latitude                    : 41 deg 15' 56.00" N
GPS Longitude                   : 95 deg 55' 20.00" W
GPS Position                    : 41 deg 15' 56.00" N, 95 deg 55' 20.00" W

```


To get the longitude and latitude I went to an online converter. (https://www.gps-coordinates.net/gps-coordinates-converter)



## **Solution and Flag Retrieval**
Final Exploit / Command Used: converted the GPS Position from exiftool to decimal degrees

![GPS coordinates converter showing the location](Where_In_The_World_1.png)
## **Lessons Learned**
- GPS coordinates can hide in metadata
- the User Comment can confirm the location: "one foot in each state" => right on the border between two states (here Nebraska and Iowa)

## **References**
- exiftool (https://exiftool.org)
- GPS coordinates converter (https://www.gps-coordinates.net/gps-coordinates-converter)
