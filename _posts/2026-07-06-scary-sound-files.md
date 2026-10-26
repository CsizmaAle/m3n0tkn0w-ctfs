---
title: "UVT CTF – Scary Sound Files"
date: 2026-07-06 08:03:00 +0200
categories: [CTF, UVT CTF]
tags: [steganography, audio, spectrogram, audacity, md5, hashing, wav]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Scary Sound Files  
**Category**:  steganography  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**  easy  
**Tags:**  steganography, audio, spectrogram, Audacity, MD5, hashing, WAV  
**Date solved:**  06/07/2026 (DD/MM/YYYY)  
**Challenge Description**:  We caught a suspicious file from a network traffic dump on an upload form. It seems to be an audio file but it's an annoyingly loud sound. We can't figure out what it is or what it's supposed to mean. Maybe you can help.  
**Files provided**:  v8q2nud9p0.wav  

## **Environment and Tools** 
**Operating System**:  macOS, Kali Linux  
**Tools/ Software**:  Audacity, Mozilla, dcode  
**Programming Languages (if any)**:  -  

## **Recognition Pattern**
- A suspicious audio file that's just "annoyingly loud" noise with no tune → the noise is almost always hiding a picture or text in the spectrogram, so switch to spectrogram view
- Once something readable shows up that names an algorithm (here MD5), treat that as the recipe for building the flag

## **Approach and Methodology**
### **Initial Analysis**
I first checked the file with `binwalk` and `file` on Kali to see if anything weird came up, but nothing did — so I went to Audacity.

### **Steps Taken**
1. I opened Audacity on macOS (it doesn't work on my Kali yet, something I need to fix), loaded the file, and switched the track to **Spectrogram**:

![the spectrogram showing a ghost and the flag recipe](Scary_Sound_Files_1.png)

A ghost and, next to it, the message that builds the flag:

```python
x = "Boo"
flag = MD5(x)
flag = 'UVTICS{' + flag + '}'
```

So the flag is `UVTICS{` + MD5 of the string `"Boo"` + `}`.

2. I wasn't sure what MD5 was at first — a hint from the teacher was that it's a hashing algorithm, so I looked it up on dcode and hashed the word `Boo`:

![dcode computing MD5 of "Boo"](Scary_Sound_Files_2.png)

That gave me the MD5 of `Boo` (the hash is the one shown in the screenshot above).

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:* put the flag together from the spectrogram recipe and the MD5 hash — `UVTICS{` + MD5 of `Boo` + `}` (the assembled flag is kept out of this write-up).

## **Lessons Learned**
- When I see something I don't recognise, like `MD5`, it's often an encryption or hashing algorithm — a quick search (or a nudge from the teacher) is usually enough to place it.
- Get the tools working *before* the event: Audacity not running on my Kali cost me a detour to macOS. Worth fixing.
- Online steganography tools aren't great — Audacity's spectrogram view did the job here where the web ones didn't.

## **References**
- <https://www.dcode.fr/en>
- Audacity
