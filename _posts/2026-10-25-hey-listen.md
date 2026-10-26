---
title: "RSTCON 2026 – Hey Listen"
date: 2026-10-25 08:10:00 +0200
categories: [Events, RSTCON 2026]
tags: [osint, social-media, linkedin, hidden-message, first-letters]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Hey Listen  
**Category**:  OSINT  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  OSINT, social-media, LinkedIn, hidden-message, first-letters  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  Hey, Hey Listen. Hey. Did you see that post about RSTCON having a CTF??  
**Files provided**:  -  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  Mozilla, LinkedIn  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- The description talks about a post announcing the CTF => look at the event's social media posts
- The post is wrapped in { } and says "read this message more carefully" => a hidden message inside the text (here the first letter of each line)

## **Approach and Methodology**
Since the description was talking about a post, i decided to open linkedin and look for a weird formatted post. 

![RSTCON LinkedIn post announcing the CTF](Hey_Listen_1.png)



## **Solution and Flag Retrieval**
Final Exploit / Command Used: -  
To put the flag together, I took the first letter from each line (the ones written in uppercase) and combined it with rstcon{}.

## **Lessons Learned**
- sometimes in OSINT challenges, CTF events, they might post a useful post long before the event starts.

## **References**
- LinkedIn (https://www.linkedin.com)
- RSTCON CTF (https://ctf.rstcon.org/)
