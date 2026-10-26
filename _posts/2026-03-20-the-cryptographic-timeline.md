---
title: "UVT CTF – The Cryptographic Timeline"
date: 2026-03-20 08:01:00 +0200
categories: [Platform, UVT CTF]
tags: [caesar-cipher, rot, vigenere, base64, xor, aes-ecb, multi-layer-encryption, cyberchef, dcode]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  The Cryptographic Timeline  
**Category**:  cryptography  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    easy  
**Tags:**  Caesar-cipher, ROT, Vigenere, Base64, XOR, AES-ECB, multi-layer-encryption, CyberChef, dcode  
**Date solved:**    20/03/2026 (DD/MM/YYYY)  
**Challenge Description**:  Welcome to The Cryptographic Timeline! We've intercepted a strange message that seems to be encrypted using a method straight out of ancient Rome. Our analysts believe this message is a Matryoshka doll of ciphers, taking you on a journey through the history of cryptography from the Caesars to the modern internet. Can you peel back the layers and find the flag?  
**Files provided**:  a text file with the ciphertext (shown below)  

## **Environment and Tools** 
**Operating System**:  Windows  
**Tools/ Software**:  dcode, ChatGPT, CyberChef, Google  
**Programming Languages (if any)**:  -  

## **Recognition Pattern**
- "Ancient Rome" / "Caesar" → a ROT/Caesar shift; in CyberChef that's the ROT13 op with an adjustable amount
- "Resisted cryptanalysis for 300 years" + "Babbage" → Vigenère, with the key hinted in the text (his last name)
- A long block of alphanumerics ending in `=`/`==` with no spaces → Base64, which is encoding, not encryption
- "Matryoshka doll" / "peel back the layers" → stacked ciphers, and each decrypted layer tells me the next cipher and key

## **Approach and Methodology**
### **Initial Analysis**
I opened the file in my browser and realized it was a ciphertext that needed to be decrypted to get the flag. Most likely I'd be using some decryption tools.  

The text inside the file:

```
Decrypt the message to find the flag.

--- MESSAGE START ---
Kszqcas hc Hvs Qfmdhcufodvwq Hwaszwbs, hfojszsf!
Ks psuwb wb obqwsbh Fcas. Xizwig Qosgof igsr o gwadzs gipghwhihwcb qwdvsf hc dfchsqh vwg awzwhofm asggousg.
Hvs bslh zomsf igsg o qwdvsf hvoh fsgwghsr qfmdhobozmgwg tcf cjsf 300 msofg, twbozzm pfcysb pm Qvofzsg *Poppous*.
Igs vwg zogh boas (iddsfqogs) og hvs ysm tcf hvs bslh zomsf.
Vsfs wg hvs zomsf 2 qwdvsfhslh:
[layer 2 ciphertext ...]
Iws hwmh apmyj xg Qpgy64 wcqdssx.
Ztft xg nzt zpnsl 3 vphp:
[long Base64 blob ...]
--- MESSAGE END ---
```

### **Steps Taken**
Since it was clearly a ciphertext (easy to tell from the category), I knew there was a website you could use to decipher text, but couldn't remember the name — so I checked Classroom for the resources posted under the assignment.  

I recognized the interface (<https://gchq.github.io/CyberChef/>), opened it, and searched for the Caesar cipher, as it was covered in the first lab and lecture. I couldn't find it by that name, tried "Shift Decoder" with no luck, so I asked ChatGPT for the CyberChef equivalent of the Caesar cipher and it pointed me to ROT13. I played with the rotation amount and found the sweet spot at **12**:  

![CyberChef ROT13 with rotation 12 decoding layer 1](Cryptographic_Timeline_1.png)

The first part read:

```
Welcome to The Cryptographic Timeline, traveler!
We begin in ancient Rome. Julius Caesar used a simple substitution cipher to protect his military messages.
The next layer uses a cipher that resisted cryptanalysis for over 300 years, finally broken by Charles *Babbage*.
Use his last name (uppercase) as the key for the next layer.
Here is the layer 2 ciphertext:
[layer 2 ciphertext ...]
```

I looked through the lecture material for the name and found nothing, so I went online. It turned out "Babbage" is associated with the Vigenère cipher, so I tried that in CyberChef — first with the original message (gibberish), then with the output from the Caesar step, and it worked. The second part read:

```
Excellent. During the Renaissance, polyalphabetic ciphers emerged to defeat frequency analysis.
The Vigenere cipher, though misattributed to Blaise de Vigenere, was considered 'le chiffre indechiffrable'.
Fast forward to the modern internet, Base64 encoding is everywhere. It's not encryption, just encoding!
The next layer is Base64 encoded.
Here is the layer 3 data:
[Base64 blob ...]
```

So I searched for Base64, added the text from the last step, and got:  

![CyberChef decoding the Base64 layer](Cryptographic_Timeline_2.png)

```
You survived the early computers! In 1917, Gilbert Vernam invented the Vernam Cipher,
which later evolved into the repeating-key XOR cipher.
To crack the next layer, use the XOR cipher with the year Vernam invented it as the key (4 digits).
Here is the layer 4 ciphertext (hex):
665c5d545e54541745561143595c115a5e5d54455f19544550181163595c...
```

As instructed, I searched for the XOR cipher and entered the key and the text. I tried different interpretations of the key (binary, hex, byte style `01 09 01 07`, ASCII key), but nothing worked in CyberChef.  

![failed XOR attempts in CyberChef](Cryptographic_Timeline_3.png)

So I switched to dcode and got the message:

```
Welcome to the modern era! The final layer uses AES-128 in ECB mode.
The Advanced Encryption Standard (AES) was established in 2001 and remains the global standard.
Your AES key is: s3cr3t_a3s_k3y_1
Here is the final ciphertext (hex):
fbb14f0ca8ea230a13e9adfeeeda01f216028f0c8744e4a1a5c32989d7dc72a5de6d3f637402460a07f7e3db66c04ce4
```

![dcode revealing the AES-128 ECB layer](Cryptographic_Timeline_4.png)

### **Alternative Approaches (if applicable)**
Other alternatives I tried included using only CyberChef or only dcode, but each failed at some point. dcode shortened the message and I got stuck at the XOR cipher; in CyberChef I failed to get the XOR step to work.  

## **Solution and Flag Retrieval**
I went back to CyberChef after finding the last ciphertext, searched for the AES Decrypt operation, entered the info from the last step, and got the flag.  

![CyberChef AES Decrypt revealing the flag](Cryptographic_Timeline_5.png)

*Final Exploit / Command Used:* AES-128 ECB decrypt in CyberChef with key `s3cr3t_a3s_k3y_1` on the final hex ciphertext.  

## **Lessons Learned**
- The main thing: each layer literally told me what cipher and key came next, so I just had to read the output carefully before moving on.
- In CyberChef the Caesar cipher is the "ROT13" op with an adjustable rotation — I wasted time searching "Caesar" and "Shift" before ChatGPT pointed me at ROT13 (rotation 12 was the sweet spot).
- Vigenère only worked once I fed it the output of the Caesar step, not the original ciphertext.
- XOR was the step that broke CyberChef for me no matter which key format I tried — switching to dcode is what got that layer.
- No single tool did the whole chain: CyberChef choked on XOR, dcode shortened the message earlier and left me stuck. Having both is what got me through.

## **References**
- <https://www.dcode.fr/en>
- <https://gchq.github.io/CyberChef/>
- <https://chatgpt.com/>
- Introduction to Cybersecurity — Cryptography course material (posted on Classroom)
