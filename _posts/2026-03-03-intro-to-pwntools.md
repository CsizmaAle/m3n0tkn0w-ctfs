---
title: "UVT CTF – Intro to pwntools"
date: 2026-03-03 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [pwntools, scripting, automation, regex, math-challenge, remote-server, speed-challenge]
---

## **Challenge Information**
**Title:**  Intro to pwntools  
**Category**:  binary exploitation (pwn)  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    easy  
**Tags:**  pwntools, scripting, automation, regex, math-challenge, remote-server, speed-challenge  
**Date solved:**    03/03/2026 (DD/MM/YYYY)  
**Challenge Description**:  Install pwntools with `pip install pwntools`. If you are on a Debian-based distribution (like Ubuntu) and `pip install pwntools` does not work, try `sudo apt install python3-pwntools`. pwntools is natively designed for Linux; it may run on macOS with some configuration but does not support Windows natively — Windows users must use a VM or WSL2.  
**Files provided**:  template.py, connection to a server  

```python
#!/usr/bin/env python3
from pwn import *

def solve():
    conn = remote('host', 1337)
    # Your implementation here
    conn.interactive()

if __name__ == "__main__":
    solve()
```

## **Environment and Tools** 
**Operating System**:  Linux  
**Tools/ Software**:  notepad, Mozilla, ChatGPT, VirtualBox  
**Programming Languages (if any)**:  Python  

## **Recognition Pattern**
- "Solve X problems in Y seconds" → way too fast for a human, so I need a script that connects, parses, and answers automatically
- The challenge handing me a pwntools template → they expect `remote()`, `recvline()`, `sendline()`
- A server firing math expressions that need instant answers → parse with regex, evaluate, send back

## **Approach and Methodology**
### **Initial Analysis**
Opening the template made it obvious I needed to write Python that connects to the given server, using the new tool we were introduced to: pwntools.  

### **Steps Taken**
First I wrote (with ChatGPT) code that just connects to the server to see what it wanted. Running it showed the CTF was about using pwntools to solve 100 math exercises in 10 seconds — something only a computer can do. So I got to work having the AI write the code, and it turned out to be surprisingly easy.

```python
#!/usr/bin/env python3
from pwn import *
import re

def solve():
    conn = remote('194.102.62.166', 30019)
    while True:
        line = conn.recvline(timeout=2)
        if not line:
            break  # if nothing more comes in, exit
        line = line.decode(errors='ignore').strip()
        print(line)
        # look for the math expression
        match = re.search(r'What is:\s*(.*)\s*\?', line)
        if match:
            expr = match.group(1)
            try:
                result = eval(expr)
            except Exception as e:
                print("Calc error:", e)
                result = 0
            print("Answer:", result)
            conn.sendline(str(result).encode())
    # print whatever comes at the end
    print(conn.recvall(timeout=2).decode(errors='ignore'))
    conn.close()

if __name__ == "__main__":
    solve()
```

### **Alternative Approaches (if applicable)**
Another way would be to write the code manually, but that could take longer — especially since I'd never worked with the pwn library before.  

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:*
```
python script.py
```

## **Lessons Learned**
- The loop for a speed challenge is just receive → parse → compute → send. Once I had that, adapting it is only a matter of changing the regex and the computation.
- `recvline()` reads a line, `recvuntil()` waits for a specific string, `sendline()` sends with a newline — those three were the core of the whole pwntools interaction.
- `eval()` was fine here for evaluating the math string, but it's dangerous on untrusted input — I'd never use it in real code, `ast.literal_eval()` is the safer option.
- The regex `r'What is:\s*(.*)\s*\?'` is what pulled the expression out of each line — basic regex keeps paying off across categories.

## **References**
- <https://chatgpt.com/>
