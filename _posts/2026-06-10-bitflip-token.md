---
title: "UVT CTF – Bitflip Token"
date: 2026-06-10 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [stream-cipher, bit-flipping, malleability, known-plaintext, xor-keystream, token-forgery, pwntools]
---

## **Challenge Information**
**Title:**  Bitflip Token  
**Category**:  cryptography  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**  medium  
**Tags:**  stream-cipher, bit-flipping, malleability, known-plaintext, XOR-keystream, token-forgery, pwntools  
**Date solved:**  10/06/2026 (DD/MM/YYYY)  
**Challenge Description**:  
**Files provided**:  server.py  

## **Environment and Tools** 
**Operating System**:  Kali Linux  
**Tools/ Software**:  VS Code  
**Programming Languages (if any)**:  python  

## **Recognition Pattern**
- The server hands out an encrypted token (`user=<name>;admin=false;expires=soon`) and gives the flag on `CHECK` only if the decrypted token contains `admin=true` → I need a token that decrypts to an admin session
- The encryption is a stream cipher — XOR against a SHA-256 keystream — with **no MAC / integrity check** → the ciphertext is malleable, I can tamper with it without ever knowing the key
- The plaintext is a fixed, known structure → I know exactly which ciphertext bytes line up with `false`, so I can flip just those

## **Approach and Methodology**
### **Initial Analysis**
I opened `server.py`. `TOKEN <name>` gives me an encrypted token of `user=<name>;admin=false;expires=soon`, and `CHECK <token>` decrypts it and prints the flag if a field reads `admin=true`. I don't have `KEY`, so I can't forge a token from scratch — but the cipher is just my message XOR'd with a keystream, and nothing authenticates the token, so it's malleable.

The relevant part of the server:

```python
FLAG = os.environ.get("FLAG", "UVTICS{fake_flag}")
KEY  = secrets.token_bytes(16)

def stream(nonce, length):        # keystream = SHA256(KEY + nonce + counter)
    ...

def encrypt(message):
    nonce = secrets.token_bytes(8)
    ciphertext = xor_bytes(message, stream(nonce, len(message)))
    return f"{nonce.hex()}.{ciphertext.hex()}"

# TOKEN -> encrypt(f"user={name};admin=false;expires=soon")
# CHECK -> if b"admin=true" in decrypt(token).split(b";"): send FLAG
```

### **Steps Taken**
Because `ciphertext = message XOR keystream`, flipping a bit of the ciphertext flips the **same** bit of the decrypted plaintext — and I already know the plaintext (`user=guest;admin=false;expires=soon`). So at the byte positions where `false` sits, I XOR in `ord(current) ^ ord(target)` to rewrite `false` → `true;`, which makes one field read `admin=true`. Then I send the forged token to `CHECK`.

```python
from pwn import *

conn = remote('HOST', PORT)

# 1. get a token for a normal user
conn.recvuntil(b'> ')
conn.sendline(b'TOKEN guest')
token = conn.recvline().strip().decode()

# 2. split nonce.ciphertext
nonce_hex, cipher_hex = token.split('.')
cipher = bytearray.fromhex(cipher_hex)     # mutable

# 3. flip the bytes of "false" -> "true;"  (known-plaintext bit-flip)
flips = [(17,'f','t'),
         (18,'a','r'),
         (19,'l','u'),
         (20,'s','e'),
         (21,'e',';')]
for pos, cur, des in flips:
    cipher[pos] ^= ord(cur) ^ ord(des)

# 4. rebuild the forged token and check it
forged = nonce_hex + '.' + cipher.hex()
conn.recvuntil(b'> ')
conn.sendline(b'CHECK ' + forged.encode())
print(conn.recvline().decode())            # the flag
conn.interactive()
```

The five flips hit the bytes of `false` (positions 17–21 in `user=guest;admin=false;...`), turning them into `true;` so a field reads `admin=true`.

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:* `python3 solve.py` — `CHECK` on the forged token returns the flag.

## **Lessons Learned**
- A stream cipher (XOR against a keystream) with no integrity check is **malleable**: if you know the plaintext, you can flip any byte to whatever you want without ever knowing the key. That's the entire attack.
- Knowing the exact token layout (`user=...;admin=false;...`) is what makes it surgical — I only had to touch the 5 bytes of `false`.
- The lesson the challenge is teaching: authenticate the token (a MAC or an AEAD cipher) so tampering gets rejected.

## **References**
none used
