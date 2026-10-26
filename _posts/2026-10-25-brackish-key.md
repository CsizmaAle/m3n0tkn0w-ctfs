---
title: "RSTCON 2026 – Brackish Key"
date: 2026-10-25 08:00:00 +0200
categories: [CTF, RSTCON 2026]
tags: [xor, repeating-key-xor, known-plaintext, crib, hex, python]
---

## **Challenge Information**
## **Challenge Information**
**Title:**  Brackish Key  
**Category**:  Cryptography  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  XOR, repeating-key-XOR, known-plaintext, crib, hex, python  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**: We pulled a note off a SecurityShrimp engagement box. Whoever wrote it did a little something to it.  
**Files provided**:  cipher.txt  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  Cyberchef  
**Programming Languages (if any)**:  bash, python  

## **Recognition Pattern**
- A hex string that decodes to gibberish => it was "done something to" after, most likely XOR
- An interesting word in the description (SecurityShrimp) that could be inside the plaintext => use it as a crib

## **Approach and Methodology**
I opened the file provided to see the ciphertext.
```bash
3117521b170b06483d0d101b5c1e45041b540201421c5e1a0058524301110306540a45161a544e000c155009000f175f1a450100540a1642065e000c051a4540450f13580011071c500006075245010e071c1107164200421a060d1c4a0d1753106e0a17031556070b052d4506003d10430f06091b42063a0917481345061d11000a16525d0b0414171107114201581a110b1c564e0a0c524506004205581c00421c54161142065803004c

```

On first seeing it, my first thought was hex, so i went on cyberchef and gave it a try. Unfortunately the output looked gibberish. I am quite positive that it is hex, so i went back to the challenge description and got the word "SecurityShrimp" which looked interesting. 

At this point, as i had no idea what to do, I decided to ask Claude.ai for some guidance. He suggested that on the result of the hex i try to find a key using the crib "SecurityShrimp" and apply XOR as it is a widely used method of encrypting messages.

That's exactly what i did.

```python3
#!/usr/bin/env python3
import binascii

cipher="3117521b170b06483d0d101b5c1e45041b540201421c5e1a0058524301110306540a45161a544e000c155009000f175f1a450100540a1642065e000c051a4540450f13580011071c500006075245010e071c1107164200421a060d1c4a0d1753106e0a17031556070b052d4506003d10430f06091b42063a0917481345061d11000a16525d0b0414171107114201581a110b1c564e0a0c524506004205581c00421c54161142065803004c"

unhexed=binascii.unhexlify(cipher)
crib = b"SecurityShrimp"   
key = [0] * 5

for i in range(len(crib)):
    key[i % 5] = crib[i] ^ unhexed[i]

plaintext = "".join(chr(unhexed[i] ^ key[i % 5]) for i in range(len(unhexed)))
print(plaintext)
print("cheie:", "".join(chr(k) for k in key))  
```

In short, the code uses the crib and the unhexed message to find the key and then applies the key to the unhexed to find the message.

To run it i had to change the solver.py permissions by using: chmod +x solver.py.
## **Solution and Flag Retrieval**
Final Exploit / Command Used: ./solver.py
```bash
SecurityShrimp field note: rotated the engagement creds tonight. maintenance token is [FLAG] do not leave it sitting on the wire next time.
cheie: br1ne
```

## **Lessons Learned**
- when there is an interesting word it can be used as a crib to find the key (XOR)
- crib XOR message = key
- the crib has to be at least as long as the key, otherwise only part of the key is found

## **References**
- Cyberchef (https://gchq.github.io/CyberChef/)
- Claude.ai (https://claude.ai)
