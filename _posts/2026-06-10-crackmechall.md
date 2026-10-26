---
title: "UVT CTF – CrackmeChall"
date: 2026-06-10 08:01:00 +0200
categories: [Platform, UVT CTF]
tags: [reverse-engineering, crackme, xor, nibble-swap, bit-rotation, rol/ror, ghidra, memcmp]
---

## **Challenge Information**
**Title:**  CrackmeChall  
**Category**:  reverse engineering  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**  medium  
**Tags:**  reverse-engineering, crackme, XOR, nibble-swap, bit-rotation, rol/ror, Ghidra, memcmp  
**Date solved:**  10/06/2026 (DD/MM/YYYY)  
**Challenge Description**:  
**Files provided**:  crackme (executable)  

## **Environment and Tools** 
**Operating System**:  Kali Linux  
**Tools/ Software**:  Ghidra, Terminal, VS Code  
**Programming Languages (if any)**:  bash, python, C  

## **Recognition Pattern**
- An executable that prompts "Enter the flag" and only answers "Correct!" / "Nope." without ever printing the flag → a crackme; reverse the check and undo its transforms
- `strings` shows those prompts plus a stray `ICS!` (a chunk of the flag format) → confirms it's a flag checker with the flag processed inside
- A fixed length check (`0x1c` = 28) → I already know the exact flag length

## **Approach and Methodology**
### **Initial Analysis**
`binwalk` and `file` gave nothing interesting (expected for a crackme), so I ran `strings` and got:

```bash
ICS!
Enter the flag: 
Nope.
Correct! That is the flag.
```

The stray `ICS!` is a piece of the flag format, which told me there's a flag being checked and some operations run on my input — either the flag has stuff done to it, or a stored string is transformed into the flag.

### **Steps Taken**
1. Opened it in Ghidra (new project → import → analyze) and read `main`:

```c
undefined8 main(void)
{
  ...
  if (lenght == 0x1c) {
    for (i1 = 0; i1 < 0x1c; i1 = i1 + 1) {
      bVar1 = input[i1 + 1] ^ *(byte *)((long)&key + (ulong)((uint)i1 & 3));
      uVar2 = rol8((bVar1 >> 4 | bVar1 << 4) - (char)i1, 3);
      local_b8[i1] = (char)uVar2;
    }
    iVar3 = memcmp(local_b8, &cipher, 0x1c);
    if (iVar3 == 0) { puts("Correct! That is the flag."); }
    else            { puts("Nope."); }
  }
  ...
}
```

So each input byte goes through, in order: **XOR with a 4-byte repeating key → nibble swap → subtract the index `i` → rotate-left by 3**, and the result is `memcmp`'d against a hardcoded `cipher`. The `key` and `cipher` are sitting in the binary:

```c
cipher = [0x0e,0x82,0x73,0x1c,0xe4,0xe7,0xe3,0xe8,0xdd,0xd7,0x43,0x30,0xb1,0xa5,
          0xaf,0x97,0x1c,0x0d,0xa7,0x11,0x6a,0x72,0xed,0xe9,0xfb,0xcd,0x67]
key    = [0x49,0x43,0x53,0x21]
```

Handy Ghidra trick to lift the cipher without retyping every byte: select the bytes → set them as a byte array → **Copy Special → C Array**.

2. From there it's just undoing the pipeline in reverse order to turn `cipher` back into the flag.

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:* `python3 script.py` — inverts the transforms and prints the flag.

```python
cipher = [0x0e,0x82,0x73,0x1c,0xe4,0xe7,0xe3,0xe8,0xdd,0xd7,0x43,0x30,0xb1,0xa5,
          0xaf,0x97,0x1c,0x0d,0xa7,0x11,0x6a,0x72,0xed,0xe9,0xfb,0xcd,0x67]
key = [0x49,0x43,0x53,0x21]

n = len(cipher)
out = []
for i in range(n):
    ch = ((cipher[i] >> 3) | (cipher[i] << 5)) & 0xFF   # undo rol3  -> ror3
    ch = (ch + i) & 0xFF                                 # undo -i    -> +i
    ch = ((ch >> 4) | (ch << 4)) & 0xFF                  # undo nibble swap 
    ch = (ch ^ key[i & 3]) & 0xFF                        # undo XOR key
    out.append(ch)

print("".join(chr(c) for c in out))
```

The inverse mirrors the forward transform in reverse order: `ror3` (undo `rol3`) → `+i` (undo `-i`) → nibble swap (its own inverse) → XOR key.

## **Lessons Learned**
- Same crackme shape as the earlier one: input → transforms → `memcmp` against a target. Find the target, undo each step in reverse order.
- Order matters — the **last** op applied (`rol3`) is the **first** one I undo (`ror3`); nibble swap and XOR are their own inverse.
- Mask with `& 0xff` on every step so the byte math stays in range.
- Ghidra's **Copy Special → C Array** is the fast way to lift the `cipher` and `key` out of the binary instead of copying bytes by hand.

## **References**
none used
