---
title: "UVT CTF – Secure Profile Manager"
date: 2026-03-22 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [aes, ecb, block-cipher, cut-and-paste-attack, pkcs7-padding, profile-manipulation, netcat]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Secure Profile Manager  
**Category**:  cryptography  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    hard  
**Tags:**  AES, ECB, block-cipher, cut-and-paste-attack, PKCS7-padding, profile-manipulation, netcat  
**Date solved:**    22/03/2026 (DD/MM/YYYY)  
**Challenge Description**:  We built an ultra-secure profile manager that uses AES encryption! Create your profile and receive an encrypted access token. Only admins can see the flag, and we made sure nobody can tamper with the role field... right?  
**Files provided**:  code files, connection to the server  

## **Environment and Tools** 
**Operating System**:  Linux (VirtualBox + Kali Linux)  
**Tools/ Software**:  Visual Studio Code, Google, Terminal , VS Code  
**Programming Languages (if any)**:  bash , python  

## **Recognition Pattern**
- The code used AES in ECB mode → each 16-byte block is encrypted independently, so identical plaintext blocks give identical ciphertext
- My input gets encrypted into a token and later decrypted to check my role → the classic ECB cut-and-paste setup
- The `&` and `=` sanitization killed injection, which pushed me toward the encryption mode as the only place a mistake could actually live
- I control part of the plaintext and need to flip `role=user` to admin → line up my input so `admin` lands in its own block, then swap blocks

## **Approach and Methodology**
### **Initial Analysis**
I opened the file provided (the server code) and started reading it, hoping to work out what I was supposed to do. After the initial read: the program asks you to create a profile and then gives you an encrypted token and user permissions, but to get the flag you need admin permissions. Since this CTF is under Cryptography, I'd have to break the token generator somehow and obtain a custom profile with admin permissions.  

### **Steps Taken**
After reading the code, I ran it locally to check whether my initial thoughts were correct, and they were.  

![running the profile manager locally](Secure_Profile_Manager_1.png)

Digging further into the code, I realized injection was off the table because there's a function that sanitizes the input:

```python
def profile_for(email: bytes) -> bytes:
    """Build a structured profile string. Sanitizes & and = to prevent injection."""
    email = email.replace(b"&", b"").replace(b"=", b"")
    return b"email=" + email + b"&role=user"
```

I kept looking for vulnerabilities, hoping to find a small door that would give me the permissions I needed. The analysis led me to the encryption/decryption mechanism, as it was the only place where a mistake could create a vulnerability.  

Not long after, I found my way in. The vulnerability was in the encryption function: it used AES in ECB mode, the least secure mode, since it encrypts each plaintext block with the same key. So to break the program I had to build a profile with a certain string, find the block that contains the role, change the token, and log in with the modified token.  

The flow of the program is: the user inputs the profile email, it gets cleaned of `&` and `=`, then padding and role are appended and it gets encrypted, so that when decrypted the role and email are easy to read.  

Since this encryption is block-based, all I have to do is create two profiles, find the block that contains the role, and swap it with one that contains the word `admin`. The profile used to create the admin block needs a specific structure: `A * 10` so that the first block is `email=AAAAAAAAAA` and the second block is `admin ` + padding.  

I used Linux for the command and the interaction with the server:

```bash
(printf "1\nAAAAAAAAAAadmin\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\n1\nBBBB\n"; cat) | nc 194.102.62.166 25704
```

The command connects to the server and creates two profiles: `AAAAAAAAAAadmin\x0b...` and `BBBB`. The `\x0b` bytes fill the rest of the block so one block is occupied only by `admin` plus that padding.

```
Block 1: email=AAAAAAAAAA         (10 A's + "email=" = 16 bytes)
Block 2: admin\x0b\x0b...          (admin + 11 PKCS7 bytes = 16 bytes)
Block 3: &role=user\x06...         (trailing)
```

After getting the tokens for both profiles, all that was left was to fabricate the final token by taking the first block of the `BBBB` profile and the middle block from the `admin` profile:

```
Profile: AAAAAAAAAAadmin\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b
Token:
  0fb8da6496d48cb97506d2fab2900f3
  975f2c766cd60a4c88739272abf27faa   # part used
  270fa019e5be44aa9eaff266c7821775

Profile: BBBB
Token:
  57b7c16a1135e9823ae257873561afaf   # part used
  0cc0b581bfcdf615a2f5ff3605b8e7f0
```

Afterwards, all that was left was to log in with the fabricated token.  

### **Alternative Approaches**
I tried Windows and Python scripts for this CTF, but for some reason all my attempts failed and I couldn't get the flag. I tried brute force (`A*something + admin +/- \x0b * something2`) and manual input but couldn't make it work — that's when I switched to Linux and bash.  

*Update:*  
I did the ctf again some time later and i came up with a python script that does all the work, all that's left is to run it:  

```python
from pwn import *

#  nc 194.102.62.183 20932
conn=remote('194.102.62.183',20932)


# first account with  admin
email= b"A" * 10 + b"admin" + b"\x0b" * 11

print(conn.recvuntil(b'> '))
conn.sendline(str(1).encode())

print(conn.recvuntil(b'Enter your email: '))
conn.sendline(email)
print(str(email))

print(conn.recvuntil(b'Your encrypted token: '))
token1=conn.recvline().decode().strip()
print(str(token1))

#second normal account 
print(conn.recvuntil(b'> '))
conn.sendline(str(1).encode())

email=b"norm"
print(conn.recvuntil(b'Enter your email: '))
conn.sendline(email)
print(str(email))

print(conn.recvuntil(b'Your encrypted token: '))
token2=conn.recvline().decode().strip()
print(str(token2))

#create admin token
role=token1[32:64]
normal=token2[:32]
admin_token=normal+role


#login as admin
print(conn.recvuntil(b'> '))
conn.sendline(str(2).encode())

print(conn.recvuntil(b'Enter your token: '))
conn.sendline(admin_token.encode())

conn.interactive()

#print(conn.recvall().decode(errors="ignore"))
#conn.close()

```


## **Solution and Flag Retrieval**
*Final Exploit / Command Used:* the fabricated token (first block of BBBB + admin block):
```
57b7c16a1135e9823ae257873561afaf975f2c766cd60a4c88739272abf27faa
```

![the server returning the flag for the admin token](Secure_Profile_Manager_2.png)

## **Lessons Learned**
- The core idea: ECB encrypts each 16-byte block on its own with the same key, so I can rearrange or swap whole blocks and it still decrypts. That's the whole weakness I abused.
- The plan that worked: pad my input so `admin` sits alone in one clean 16-byte block, then splice that block into another token in place of the original role.
- The `\x0b` × 11 padding is just PKCS7 filling the rest of the block (16 − 5 = 11), which is what makes `admin` occupy a clean block.
- `printf` + `nc` in bash gave me exact byte-level control over the input. After my Windows and Python attempts kept failing for no clear reason, that's what finally worked.

## **References**
- <https://www.geeksforgeeks.org/computer-networks/advanced-encryption-standard-aes/>
- <https://www.geeksforgeeks.org/computer-networks/electronic-code-book-ecb-in-cryptography/>
- <https://chatgpt.com/>
