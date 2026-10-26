---
title: "UVT CTF – Walking through time..."
date: 2026-07-06 08:00:00 +0200
categories: [Platform, UVT CTF]
tags: [multi-layer, base64, rot13, hex, caesar, vigenère, cyberchef, dcode, matryoshka]
---

## **Challenge Information**
**Title:**  Walking through time...  
**Category**:  cryptography  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**  easy  
**Tags:**  multi-layer, Base64, ROT13, hex, Caesar, Vigenère, CyberChef, dcode, Matryoshka  
**Date solved:**  06/07/2026 (DD/MM/YYYY)  
**Challenge Description**:  Decrypt the message to find the flag. Each decoded layer contains a short note that points to the next step.  
**Files provided**:  challenge.txt  

## **Environment and Tools** 
**Operating System**:  Kali Linux  
**Tools/ Software**:  dcode, CyberChef, Mozilla  
**Programming Languages (if any)**:  -  

## **Recognition Pattern**
- A text file of ciphertext + "each decoded layer points to the next step" → a Matryoshka-doll of stacked ciphers, peel one layer at a time and read the note before moving on
- The first block is a long string, alphanumeric plus `+/`, ending in `=` → Base64 is the outer layer

## **Approach and Methodology**
### **Initial Analysis**
I first ran the file through `binwalk` and `file`, but nothing interesting came up, so I just opened it (I could have used `cat` too). Inside was a message wrapped in `--- MESSAGE START ---` / `--- MESSAGE END ---`:

```
--- MESSAGE START ---
TGF5ZXIgMSBjb21wbGV0ZTogQmFzZTY0LgpOZXh0IHN0ZXA6IFJPVDEzLgpBcHBseSBST1QxMyB0...
--- MESSAGE END ---
```

It looked like Base64, so I checked on dcode (which agreed) and then switched to CyberChef to work through the layers.

### **Steps Taken**
The nice thing about this one is that every layer decodes to a note telling me exactly what the next layer is:

1. **Base64** → *"Layer 1 complete: Base64. Next step: ROT13."*
2. **ROT13** → *"Layer 2 complete: ROT13. Next step: hex."* (ROT13 is just Caesar with shift 13.)
3. **hex → bytes** → *"Layer 3 complete: hex. Next step: Caesar cipher."* The note said the block was shifted forward by 14 letters, so I shifted it back by 14.
4. **Caesar (shift 14 back)** → *"Layer 4 complete: Caesar. Next step: Vigenere. Use uppercase key TIMELINE."*
5. **Vigenère (key `TIMELINE`)** → the final block:

```
FINAL NOTE
Algorithms used: Base64, ROT13, hex, Caesar cipher, Vigenere cipher.
Flag: UVTICS{...}
```

So the whole chain was **Base64 → ROT13 → hex → Caesar → Vigenère**, and the final note hands over the flag.

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:* CyberChef with a Vigenère Decode on the last block, key `TIMELINE` — the FINAL NOTE it produced hands over the flag (kept out of this write-up).

## **Lessons Learned**
- Multi-layer challenges hand you breadcrumbs — each decoded layer names the next cipher and sometimes the key, so I just read the note carefully before moving on.
- Base64 is *encoding*, not encryption — no key, just decode it — which made spotting the first layer easy.
- For the Vigenère step the whole block has to be decrypted continuously with the key (`TIMELINE`), not line by line, or only the first line comes out right.

## **References**
- <https://gchq.github.io/CyberChef/>
- <https://www.dcode.fr/cipher-identifier>
