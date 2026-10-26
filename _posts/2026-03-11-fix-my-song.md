---
title: "UVT CTF – Fix my song!"
date: 2026-03-11 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [steganography, audio, spectrogram, audacity, hidden-message]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Fix my song!  
**Category**:  steganography  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    easy  
**Tags:**  steganography, audio, spectrogram, audacity, hidden-message  
**Date solved:**    11/03/2026 (DD/MM/YYYY)  
**Challenge Description**:  My friend sent me a cool Queen song. He said the file seems corrupted, can you try to see what is wrong with it?  
**Files provided**:  an audio file  

## **Environment and Tools** 
**Operating System**:  Linux (VirtualBox + Ubuntu)  
**Tools/ Software**:  Audacity, Google, VirtualBox  
**Programming Languages (if any)**:  -  

## **Recognition Pattern**
- Audio file + "corrupted" / "something wrong" → check the spectrogram
- Glitchy sounds that don't follow any pattern → likely visual data drawn into the spectrogram, not morse code
- Check the start and end of the spectrogram first — flags are often hidden there

## **Approach and Methodology**
### **Initial Analysis**
After reading the description and seeing an audio file, I assumed it was some hidden message inside the audio, maybe using morse code. I downloaded and listened to the file and still had my suspicions. Having heard of steganography, I also considered a message hidden in the audio spectrogram.  

### **Steps Taken**
I went online to download Audacity and opened the provided audio file in it. Then I switched from Waveform to Spectrogram.  

![switching the track to Spectrogram view](Fix_My_Song_1.png)

After this change I saw a weird pattern.  

![a weird pattern visible in the spectrogram](Fix_My_Song_2.png)

### **Alternative Approaches**
I first tried listening to the audio, thinking the flag was hidden with morse code, but after a short listen it was clear that wasn't the case — there was no rule to how the weird sounds worked.  

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:* zoom in at the beginning of the spectrogram.  

## **Lessons Learned**
- Audio that sounds off but has no clear pattern (not morse, not tones) → I switch to spectrogram view straight away.
- In Audacity: open the file, click the track dropdown, pick Spectrogram — that was my first real move and where the flag showed up.
- Zoom into different parts of the spectrogram — mine was at the very start, but they can also hide at the end or in a specific frequency band.
- I burned a little time listening for morse first — worth ruling that out fast when there's no rule to the weird sounds.

## **References**
none used
