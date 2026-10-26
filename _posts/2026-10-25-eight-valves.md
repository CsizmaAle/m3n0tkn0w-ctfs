---
title: "RSTCON 2026 – Eight Valves"
date: 2026-10-25 08:01:00 +0200
categories: [Events, RSTCON 2026]
tags: [esoteric-language, brainfuck, dcode, cipher-identifier]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
## **Challenge Information**
**Title:**  Eight Valves  
**Category**:  Misc  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  esoteric-language, Brainfuck, dcode, cipher-identifier  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  We recovered a control program from a valve skid. Though, I only know Java programming. What is this?  
**Files provided**:  valves.bf  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  dcode  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- The file is made only out of `+ - .` characters (Brainfuck uses `+ - . , < > [ ]`)
- The .bf extension of the file
- The description says it's a "control program" but not in a language they know => code in some weird (esoteric) language

## **Approach and Methodology**
I opened the provided file and found what looked like a cipher, but i was not so sure about it.
```bash
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
++++++++++++++++++++++++++++++++++++++++++++++++++.+.+.---------
--------.++++++++++++.-.+++++++++++++.----------------------.+++
+.--.+.++++++++++++.---------------------.++++++++++++++++++++++
+.---------------------.+++++++++++.++++++++++.-----------------
.++++++++++++++.--------------------.+++++++++++++++++++.+++.---
----.---------------.+++++++++++++++++++++.------------.---.----
--.+++++++++++++++++++++.-------------------.+++++++++++++++.---
--------.++++++++++++++++++++++++.

```


So after reading it, i opened dcode, cipher identifier and it appeared to be Brainfuck.
![dcode cipher identifier suggesting Brainfuck](Eight_Valves_1.png)

So i went ahead and searched for it in dcode and entered the cipher again.


## **Solution and Flag Retrieval**
Final Exploit / Command Used: Execute on Brainfuck interpreter in dcode.  
![dcode Brainfuck interpreter printing the flag](Eight_Valves_2.png)

## **Lessons Learned**
- when finding a cipher or something that looks like one, one of the fastest ways to identify it is by using cipher identifier in dcode
- the existence of Brainfuck 

## **References**
- Dcode (https://www.dcode.fr/en)
- Dcode Cipher Identifier (https://www.dcode.fr/cipher-identifier)
- Dcode Brainfuck interpreter (https://www.dcode.fr/brainfuck-language)
