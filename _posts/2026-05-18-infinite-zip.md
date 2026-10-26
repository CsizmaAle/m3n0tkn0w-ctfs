---
title: "UVT CTF – Infinite ZIP"
date: 2026-05-18 08:00:00 +0200
categories: [Platform, UVT CTF]
tags: [nested-archives, automation, outlier-detection, base64]
---

## **Challenge Information**
**Title:**  Infinite ZIP  
**Category**:  forensics  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**  easy  
**Tags:**  nested-archives, automation, outlier-detection, base64  
**Date solved:**  18/05/2026 (DD/MM/YYYY)  
**Challenge Description**:  We found this file that seems to be infected with "ZIPception", please help us recover the flag hidden somewhere in here.  
**Files provided**:  an archive  

## **Environment and Tools** 
**Operating System**:  Kali Linux  
**Tools/ Software**:  Mozilla, terminal, VirtualBox, 7z, base64  
**Programming Languages (if any)**:  bash, python  

## **Recognition Pattern**
- The name "ZIPception" said it all — nested zips, Inception-style
- "hidden somewhere" is singular, so the flag's in ONE place, not something I reconstruct from pieces
- After a few layers, each one had the same structure (a blob, two notes with techy metadata, the next zip) → with 1000 layers this was a needle-in-a-haystack, not a transformation puzzle

## **Approach and Methodology**
### **Initial Analysis**
I opened the first archive manually and saw `blob_0001.bin` + `note_0001_a.txt` + `note_0001_b.txt` + `layer_0002.zip`. The notes contained things like `tag=rotate-payload`, `ref=72554`, `checksum=0e660260`, which made it look like a transformation challenge. I almost went down that path before realising the description said "hidden somewhere", which means find, not solve.  

### **Steps Taken**
1. Confirm the structure by extracting a few more layers manually:
```bash
7z x layer_0002.zip -olayer_0002
```
The tags varied per layer (`rotate-payload`, `archive-cluster`, `collect-telemetry`, `record-marker`, etc.) — classic distraction. I ignored the notes from that point.

2. Write a Python script to extract all 1000 layers and analyse for outliers:
- Walk the chain: extract each zip, find the next zip inside, repeat.
- After extraction, check every blob's size and file type.
- Look for any file that does NOT match the blob/note/zip pattern.
- Grep all extracted content for flag patterns (`flag{`, `UVTICS{`, `ctf{`).

3. Run the script:
```bash
python3 script.py Layer-master-v3204/layer_0002.zip
```

```python
#!/usr/bin/env python3
"""Extract all layers of ZIPception and identify outliers."""
import subprocess, sys, re
from pathlib import Path
from collections import Counter

def extract(archive, dest):
    dest.mkdir(parents=True, exist_ok=True)
    r = subprocess.run(['7z', 'x', str(archive), f'-o{dest}', '-y'],
                       capture_output=True, text=True)
    return r.returncode == 0

def main(start):
    work = Path('unpacked')
    work.mkdir(exist_ok=True)
    cur = work / 'layer_0000'
    extract(Path(start), cur)
    layer = 0
    while True:
        zips = list(cur.glob('*.zip'))
        if not zips:
            print(f'\n[final layer: {layer}]')
            print(f'contents of {cur}:')
            for f in sorted(cur.iterdir()):
                print(f'  {f.name} ({f.stat().st_size} bytes)')
            break
        layer += 1
        nxt = work / f'layer_{layer:04d}'
        if not extract(zips[0], nxt):
            print(f'failed at layer {layer}')
            break
        cur = nxt
        if layer % 100 == 0:
            print(f'  ...layer {layer}')

    print('\n=== OUTLIER ANALYSIS ===\n')
    # Collect every file across all layers
    all_files = list(work.rglob('*'))
    blobs = [f for f in all_files if f.is_file() and f.name.startswith('blob_')]
    notes = [f for f in all_files if f.is_file() and f.name.startswith('note_')]
    others = [f for f in all_files if f.is_file()
              and not f.name.startswith(('blob_', 'note_'))
              and not f.suffix == '.zip']

    # Size distribution of blobs — outliers stand out
    sizes = Counter(f.stat().st_size for f in blobs)
    print(f'Blob size distribution: {dict(sizes)}')
    common_size = sizes.most_common(1)[0][0]
    odd_blobs = [f for f in blobs if f.stat().st_size != common_size]
    print(f'\nBlobs with non-standard size: {len(odd_blobs)}')
    for f in odd_blobs[:20]:
        print(f'  {f} ({f.stat().st_size} bytes)')

    # File-type check on every blob — anything that isn't "data" is suspicious
    print('\nBlobs that file(1) recognizes as something:')
    for f in blobs:
        r = subprocess.run(['file', '-b', str(f)], capture_output=True, text=True)
        t = r.stdout.strip()
        if t.lower() != 'data':
            print(f'  {f}: {t}')

    # Files that don't fit the blob/note/zip pattern
    if others:
        print(f'\nUnexpected files (not blob/note/zip): {len(others)}')
        for f in others[:20]:
            print(f'  {f}')

    # Grep notes broadly — flag, ctf, base64-looking strings
    print('\nGrep results across notes:')
    for pattern in ['flag', 'ctf{', 'flag{', 'THM{', 'HTB{']:
        r = subprocess.run(['grep', '-ril', pattern, str(work)],
                           capture_output=True, text=True)
        if r.stdout.strip():
            print(f'  pattern "{pattern}":')
            print(r.stdout)

    # Look for base64-encoded strings in notes
    print('\nLong base64-looking strings in notes:')
    b64 = re.compile(rb'[A-Za-z0-9+/]{20,}={0,2}')
    for n in notes:
        data = n.read_bytes()
        for m in b64.finditer(data):
            s = m.group()
            if not s.isdigit():  # skip pure number runs
                print(f'  {n.name}: {s[:60]!r}...')
                break

if __name__ == '__main__':
    main(sys.argv[1])
```

*Update:*  
I re-did this CTF a while later and found a better solution (one actually written by me :)) )
```python
import os
from pathlib import Path
import shutil

main_directory = Path.cwd()
results     = main_directory / "results"
interesting = results / "interesting"
unarchived  = results / "unarchived"

for d in (results, interesting, unarchived):
    d.mkdir(parents=True, exist_ok=True)

current_directory = main_directory

while True:
    for item in current_directory.iterdir():
        if item.is_file() and "flag" in item.name.lower():
            shutil.copy(item, interesting / item.stem)

    archives = [p for p in current_directory.iterdir()
                if p.suffix.lower() == ".zip"]
    if not archives:
        break

    next_dir = unarchived / archives[0].stem
    next_dir.mkdir(parents=True, exist_ok=True)
    shutil.unpack_archive(archives[0], next_dir)

    current_directory = next_dir

print("Done")

```

Back to the olt writeup now :))  

The output showed one critical line under "Unexpected files (not blob/note/zip):" → `unpacked/layer_0418/flag.txt`. That was the outlier — every other layer had only blob/note/zip files. Layer 418 had an extra file called `flag.txt`.

4. Read the flag file:
```bash
cat unpacked/layer_0418/flag.txt
```
Output:
```
VVZUSUNTe0QzM3BfRDF2M180MjB9
```
This is base64 (only A-Z, a-z, 0-9, length a multiple of 4).

### **Alternative Approaches (if applicable)**
Could have gone straight to the final layer (998) and checked for the flag there — but it would have failed because the flag was at layer 418, not the end. Could have used file size as the outlier criterion instead of file name, but all blobs were 64 bytes so that wouldn't have helped here. Could have used a different scripting language (bash with a while loop and 7z) for the extraction part.

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:*
```
echo "VVZUSUNTe0QzM3BfRDF2M180MjB9" | base64 -d
```

## **Lessons Learned**
- When the description says "hidden somewhere", I look for the outlier instead of trying to solve a puzzle. With thousands of files I can't inspect each one, so I hunt for what does NOT match the pattern.
- The suspicious-looking metadata (`tag`, `ref`, `checksum`) turned out to be pure flavour to distract me — the notes were red herrings and I dropped them early.
- I almost forgot to check the format before submitting — the flag came out as base64, which decodes in one command.
- 1000 layers by hand is impossible; with a script it's one cup of coffee.
- Nice touch: the flag sat at layer 418 and its content had "420" in it, so the layer number was a little wink from the author.

## **References**
- Claude.ai
