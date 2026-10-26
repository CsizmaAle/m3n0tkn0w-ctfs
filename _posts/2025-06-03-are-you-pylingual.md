---
title: "UVT CTF – Are you PYlingual?"
date: 2025-06-03 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [reverse-engineering, pyinstaller, python-bytecode, dll-analysis, pipeline-inversion, lcg]
---

## **Challenge Information**
**Title:**  Are you PYlingual?  
**Category**:  reverse engineering  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    medium  
**Tags:**  reverse-engineering, PyInstaller, python-bytecode, DLL-analysis, pipeline-inversion, LCG  
**Date solved:**     3/06/2026 (DD/MM/YYYY)  
**Challenge Description**:  A homemade flag scrambler, frozen into one Windows executable, with only the output left behind. Unfreeze it, follow the bytes, get the flag.  
			Hint: Use Binary Ninja, Iaito, IDA, Ghidra or any other reverse engineering framework to solve this challenge.  
**Files provided**:  2 executables that seemed to be the same, and an output.txt file  

## **Environment and Tools** 
**Operating System**:  Kali Linux  
**Tools/ Software**:  file, strings, Ghidra, Binary Ninja, pyinstxtractor, pycdc/pycdas (zrax/pycdc)  
**Programming Languages (if any)**:  Python, bash  

Other methodologies: static analysis, pipeline mapping (reading the program top-to-bottom to model every transform), then inverting each transform in reverse order.  

## **Recognition Pattern**
- The description saying "frozen" and "unfreeze" is classic PyInstaller talk — that's what freezes a Python program into one executable
- The name "Are you PYlingual?" leaning on **PY** was another nudge toward Python
- `strings` on the exe was full of PyInstaller giveaways: `pyiboot01_bootstrap`, `pyimod01_archive`, `python313.dll`, `base_library.zip`, `PYZ.pyz`
- The same output listed the challenge's own files — `banner.b64`, `crypto.dll`, `challenge` — so the real logic was split between bundled Python and a custom DLL
- Put together: a PyInstaller-packed Python program with a native scramble routine in a DLL, so the path is unfreeze → recover the Python → recover the DLL → invert the whole pipeline

## **Approach and Methodology**
### **Initial Analysis**
From the description it was clear I had to reverse engineer a Python script with the right tool. I downloaded the files and ran `file` and `strings` on `challenge.exe`. The end of the `strings` output showed something interesting about some `.dll` files and a `.b64` file, alongside a long list of bundled Python modules:

```
...
base64)
ctypes)
...
pyimod01_archive
pyimod02_importers
pyimod03_ctypes
pyiboot01_bootstrap
challenge
banner.b64
base_library.zip
crypto.dll
python313.dll
pyi-contents-directory _internal
PYZ.pyz
python313.dll
```

The presence of PyInstaller bootstrap modules, `python313.dll`, `base_library.zip`, `PYZ.pyz`, plus the project-specific `banner.b64` and `crypto.dll`, confirmed this was a PyInstaller-frozen Python executable built with Python 3.13.

### **Steps Taken**
I used Ghidra to try to get something directly out of the executable; nothing useful came out for `challenge.exe`, so I tried Binary Ninja — which didn't help much either. (In hindsight this is expected: a native disassembler on a PyInstaller exe only shows the bootloader stub, not the Python logic.)  

I installed pyinstxtractor and ran it to unfreeze the executable:
```bash
python3 pyinstxtractor.py challenge.exe
```

This produced a `challenge.exe_extracted/` folder. I opened `challenge.pyc`, and the readable strings inside described the whole challenge:

> A banner has the flag stitched into it one character at a time, then the whole thing is run through the custom C routine in `crypto.dll`. The scrambled byte list lands in `output.txt`.

Other important items visible in the bytecode were the referenced files and functions: `crypto.dll`, `output.txt`, `uvt_obfuscate`, `b64decode`, `banner.b64`, `ascii`, `latin-1`, `flag.txt`, `utf-8`.  

I opened the bundled files. Base64-decoding `banner.b64` produced a slant-figlet ASCII-art banner reading **"WEST UNIVERSITY of TIMISOARA ICS CTF"** — the flag wasn't visibly spelled out, so it had to be stitched in among the art's bytes. `flag.txt` was **not** bundled (the author excluded it), which is why the flag had to be reversed out of `output.txt`.  

When I opened `crypto.dll` in Ghidra, under **Exports** I found the custom function `uvt_obfuscate` (all other exports were C-runtime boilerplate). Its decompilation revealed the scramble algorithm:

```c
void uvt_obfuscate(longlong param_1, int length_of_param_1)
{
    uint uVar1;
    undefined4 i;
    undefined4 local_c;

    local_c = length_of_param_1 * -0x61c8864f ^ 0x49545655;
    for (i = 0; i < length_of_param_1; i = i + 1) {
        local_c = local_c * 0x41c64e6d + 0x3039;
        uVar1 = rol8(~*(byte *)(i + param_1) ^ (byte)(local_c >> 0x10),
                     (char)i + (char)(i / 7) * -7);
        *(char *)(param_1 + i) = (char)uVar1 + (char)i;
    }
    return;
}
```

Reading it:
- `local_c` is a running key, seeded from the buffer length (`len * -0x61c8864f ^ 0x49545655`; note `0x49545655` is ASCII `"UVTI"` — the author's signature).
- Each iteration advances the key with a linear-congruential generator (`key = key * 0x41c64e6d + 0x3039` — the standard C `rand()` constants), so the keystream is fully reproducible.
- The rotate amount `(char)i + (char)(i/7)*-7` is just `i % 7`.

**Each byte goes through:** NOT → XOR(key >> 16) → rotate-left by `i%7` → `+ i`. **Therefore the reverse, in opposite order, is:** `- i` → rotate-right by `i%7` → XOR → NOT.  

To get readable logic for `challenge.pyc` I needed a Python 3.13-capable tool. uncompyle6/decompyle3 don't support 3.13, and pycdc could only partially decompile it (it failed on `MAKE_FUNCTION`), so I used its disassembler `pycdas`:
```bash
git clone https://github.com/zrax/pycdc
cd pycdc
cmake .
make
./pycdas ../challenge.exe_extracted/challenge.pyc > dis.txt
```

Reading the main disassembly in `dis.txt` (following the `STORE_FAST` variable names and the constants `10, 28, 128, 256, 2`), I reconstructed the full forward pipeline:

**banner → list() → stitch flag at positions `start, start+28, start+56, ...` (where `start = len(art) % 10`) → join to bytes → uvt_obfuscate (NOT→XOR→ROL→+i) → byte→signed → swap halves → save to output.txt**

The two Python-side steps that are easy to miss are the **signed conversion** (`b - 256 if b >= 128 else b`) and the **half-swap** (`out = signed[half:] + signed[:half]`, where `half = n // 2`). These happen *after* the DLL call, so they must be undone *first* when reversing.

**Inverting the pipeline**  
To recover the flag, the full forward pipeline is reversed bottom-to-top:
1. **Undo the half-swap** — restore the original byte order.
2. **Undo the signed conversion** — map signed values back to 0..255.
3. **Undo `uvt_obfuscate`** — per byte `i`: `- i` → ROR(`i%7`) → XOR(keystream) → NOT, regenerating the same LCG keystream from the length `n`.
4. **Un-stitch the flag** — read characters from positions `start, start+28, start+56, ...` where `start = len(art) % 10`.

### **Alternative Approaches**
- **Use the DLL as an oracle instead of reversing it by hand.** Because the scramble is deterministic and reversible, one could load `crypto.dll` directly (with `ctypes` on Windows, or by reimplementing `uvt_obfuscate` verbatim from the Ghidra output) and brute-force/confirm the inverse without hand-deriving it.
- **Match the Python version and use a native decompiler.** Installing CPython 3.13 and using the built-in `dis` module (or a 3.13-aware decompiler once available) would give cleaner source than raw `pycdas` bytecode.
- **Parameter brute-force.** Early on, the rotate direction and key-advance timing were uncertain; trying all combinations programmatically (rotate left/right × advance before/after × signed/unsigned subtraction) is a valid fallback when a single transform detail is ambiguous — though understanding the bytecode directly is faster and more reliable.

## **Solution and Flag Retrieval**
**Final solver script:**
```python
import ast

# Load output.txt (a Python list of SIGNED bytes)
out = ast.literal_eval(open('output.txt').read())
n = len(out)

def s32(x):
    x &= 0xFFFFFFFF
    return x - 0x100000000 if x & 0x80000000 else x

def ror8(v, r):  # undo forward rol8
    r &= 7
    return ((v >> r) | (v << (8 - r))) & 0xFF

def signed_char(i):  # emulate (char)i
    i &= 0xFF
    return i - 256 if i >= 128 else i

# STEP 1 - undo the half-swap: out = signed[half:] + signed[:half]
half = n // 2
first_len = n - half
signed = out[first_len:] + out[:first_len]

# STEP 2 - signed -> unsigned (0..255)
b = [(x + 256) & 0xFF for x in signed]

# STEP 3 - undo uvt_obfuscate: -i -> ror(i%7) -> XOR -> NOT
key = s32(s32(n * -0x61c8864f) ^ 0x49545655)
plain = bytearray(n)
for i in range(n):
    key = s32(s32(key * 0x41c64e6d) + 0x3039)
    ks = (key >> 16) & 0xFF
    v = (b[i] - signed_char(i)) & 0xFF
    v = ror8(v, i % 7)
    v ^= ks
    v = (~v) & 0xFF
    plain[i] = v

art = plain.decode('latin-1')

# STEP 4 - un-stitch the flag from positions start, +28, +56, ...
start = len(art) % 10
flag = ''
pos = start
while pos < len(art):
    flag += art[pos]
    pos += 28

print("banner preview:", repr(art[:120]))
print("FLAG:", flag)
```

Another version of the script:  

```python
import ast

output= ast.literal_eval(open("output.txt", "r").read())
n=len(output)

# swap the first half with the second one
half=n//2
first_len=n-half
signed=output[first_len:] + output[:first_len]

#undo signed convention
unsigned = bytes((x + 256) & 0xFF for x in signed)

def ror8(v,r):
    r=r&7
    return ((v >> r) | (v << (8 - r))) & 0xFF

def s32(x):
    x &= 0xFFFFFFFF
    return x - 0x100000000 if x & 0x80000000 else x

def unbfuscate(data):
    n=len(data)
    key = s32((n * -0x61c8864f) ^ 0x49545655)
    out=bytearray(n)
    for i in range(n):
        key = s32((key * 0x41c64e6d) + 0x3039)
        v=data[i]

        # undo -i
        v= (v-i) & 0xFF

        #undo rotation to left
        v= ror8(v,i%7)

        #undo XOR with key shift to 16
        v= (v ^ (key >> 16) )& 0xFF

        #undo NOT
        v=(~v) & 0xFF

        #save
        out[i]=v
    return bytes(out)

rez=unbfuscate(unsigned)

art = rez.decode('latin-1')
start = len(art) % 10
flag = ''
pos = start
while pos < len(art):
    flag += art[pos]
    pos += 28
if '}' in flag:
    flag = flag[:flag.index('}') + 1]


print("FLAG:", flag)
```


*Final Exploit / Command Used:* `python3 solve.py` — it printed the readable banner (confirming the inverse was correct) and the recovered flag.  

## **Lessons Learned**
- "Frozen" meant PyInstaller, and spotting that from the description and `strings` decided my whole toolchain (pyinstxtractor + a bytecode tool) instead of me burning time in a native disassembler.
- Running Ghidra/Binary Ninja on `challenge.exe` was a dead end — a native disassembler only shows the bootloader stub. The real logic was in the bundled Python and the separate `crypto.dll`, and those needed different tools.
- My hardest mistake: trying to undo `uvt_obfuscate` while ignoring the Python-side half-swap and signed conversion that happen *after* it. Reading `main` end-to-end and listing every transform first is what made the inverse straightforward.
- To reverse a pipeline I had to go in the exact opposite order and invert each step — XOR and NOT are their own inverse, `+i` undoes with `-i`, rotate-left with rotate-right.
- The "crypto" was fully reversible because the LCG keystream only depends on the public length `n` — there was no secret key.
- When pycdc failed to decompile, dropping to its disassembler `pycdas` got me the logic instead of me switching tools entirely.

## **References**
- pyinstxtractor — <https://github.com/extremecoders-re/pyinstxtractor>
- pycdc / pycdas (Decompyle++) — <https://github.com/zrax/pycdc>
- Ghidra — <https://ghidra-sre.org>
