---
title: "RSTCON 2026 – Carry The Tide"
date: 2026-10-25 08:04:00 +0200
categories: [CTF, RSTCON 2026]
tags: [reverse-engineering, crackme, ghidra, chained-check, hardcoded-target, python]
---

## **Challenge Information**
**Title:**  Carry The Tide  
**Category**:  Reverse Engineering  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  reverse-engineering, crackme, Ghidra, chained-check, hardcoded-target, python  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**: A sluice controller will only align if you give it the right control key.  

**Files provided**:  carry (ELF)  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  Ghidra  
**Programming Languages (if any)**:  bash, python  

## **Recognition Pattern**
- A binary that asks for a "control key" and checks it against something hardcoded => crackme, recover the key
- The check compares (running_key + input[i]) against a hardcoded target array, where the key is the previous input character => a reversible chained transform

## **Approach and Methodology**
I decided to open the file in Ghidra and analyze it. There were two interesting functions: main and check
```C

undefined4 main(void)

{
  char *controlKey;
  size_t sVar1;
  int iVar2;
  undefined4 uVar3;
  char acStack_8c [128];
  int local_c;
  
  local_c = __stack_chk_guard;
  printf("Carry the tide. Enter control key: ");
  controlKey = fgets(acStack_8c,0x80,stdin);
  if (controlKey == (char *)0x0) {
    uVar3 = 1;
  }
  else {
    sVar1 = strcspn(acStack_8c,"\r\n");
    acStack_8c[sVar1] = '\0';
    iVar2 = check(acStack_8c);
    if (iVar2 == 0) {
      puts("Out of phase.");
    }
    else {
      puts("Sluice aligned.");
    }
    uVar3 = 0;
  }
  if (__stack_chk_guard != local_c) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail(0,0);
  }
  return uVar3;
}


undefined4 check(char *param_1)

{
  size_t length_1;
  undefined4 uVar1;
  char key;
  int i;
  
  length_1 = strlen(param_1);
  if (length_1 == 0x26) {
    key = 'Z';
    for (i = 0; i < 0x26; i = i + 1) {
      if ((char)(key + param_1[i]) != target[i]) {
        return 0;
      }
      key = param_1[i];
    }
    uVar1 = 1;
  }
  else {
    uVar1 = 0;
  }
  return uVar1;
}


```

The key was the flag, and in order to obtain it i looked inside the check function. The function was simple: the input was added to a changing key and tested against a target value.

Since the first letter of the key was Z, and it changed based on the input, reversing it was pretty easy. I copied from ghidra the target (it was hardcoded) and applied - key to get the initial input (or what it should be - the flag). Also, the key is the recovered letter.

```python3
#!/usr/bin/env python3

list =[ 0xcc, 0xe5, 0xe7, 0xd7, 0xd2, 0xdd, 0xe9, 0xed, 0xe7, 0xe3, 0xdc, 0xd7, 0xd7, 0xd5, 0xc6, 0xd2, 0xe8, 0xe2, 0xcc, 0xd4, 0xe3, 0xe5, 0xe0, 0xd7, 0xd2, 0xd7, 0xd2, 0xc8, 0xd7, 0xcd, 0xd1, 0xd7, 0xdb, 0xdb, 0xd7, 0xe5, 0xd8, 0xe2 ]

text=[]
key=ord('Z')

for i in list:
	var=i-key
	text+=(chr(var))
	key=var

for e in text:
	print(e, end="")
```


## **Solution and Flag Retrieval**
Final Exploit / Command Used: ./solver.py

## **Lessons Learned**
- if the target and the key are hardcoded, the intended input can be reversed
- in a chained check where each step depends on the previous character, undo it step by step (subtract the running key, which becomes the recovered character)

## **References**
- Ghidra
