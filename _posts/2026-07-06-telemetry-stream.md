---
title: "UVT CTF – Telemetry stream"
date: 2026-07-06 08:01:00 +0200
categories: [Platform, UVT CTF]
tags: [reverse-engineering, rc4, stream-cipher, base64, strings, cyberchef, ksa-bug, red-herring]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Telemetry stream  
**Category**:  reverse engineering  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**  easy  
**Tags:**  reverse-engineering, RC4, stream-cipher, Base64, strings, CyberChef, KSA-bug, red-herring  
**Date solved:**  06/07/2026 (DD/MM/YYYY)  
**Challenge Description**:  A single native program claims to recover an archived pump-station telemetry note. Running it prints junk because the built-in stream-cipher routine has a small bug.  
			Recommended tools: IDA / Ghidra / Binary Ninja and CyberChef / dCode.  
**Files provided**:  crackme (a native binary)  

## **Environment and Tools** 
**Operating System**:  Kali Linux  
**Tools/ Software**:  Mozilla, Ghidra, CyberChef  
**Programming Languages (if any)**:  bash, C  

## **Recognition Pattern**
- A native binary that prints junk, and the description says its stream-cipher routine has a bug → the intended path is to redo the cipher *correctly* myself, not trust the program's output
- "Recover an archived note" + a stream cipher → some encoding wrapped in RC4; run `strings` before opening Ghidra, because these often leave the ciphertext, key and even hints sitting in plaintext

## **Approach and Methodology**
### **Initial Analysis**
Since it's a reverse-engineering challenge I first looked at the binary with `file` and `checksec`, but nothing useful came up, so I fell back to `strings`.

### **Steps Taken**
1. `strings crackme` basically handed me the whole challenge — the ciphertext, the key, and a set of hints spelling out exactly what to do:

```bash
vsj/x0prFIhCRtIcrkuqztvyWJTNMVfOs+WRk6EESkVaiuekL0DSn70XIP37ClRbUvmTzFTZdLz...  (long Base64 blob)
TelemetryTea
HINT: the telemetry string is Base64.
HINT: after Base64, the bytes are RC4 encrypted.
HINT: stream key = TelemetryTea.
HINT: the local decryptor is close, but its KSA starts from the wrong key byte.
HINT: use a correct RC4 implementation instead of trusting this output.
bad telemetry
== PB15 telemetry recovery console ==
```

So the "bug" the description mentioned is right there in the hints — the binary's own RC4 key-scheduling (KSA) starts from the wrong byte, which is why it prints garbage. I didn't need to fix the binary; I just had to run a correct RC4 myself.

2. I opened CyberChef and cooked a recipe from the data in the strings output: **From Base64** → **RC4** with `Passphrase = TelemetryTea` (Latin1 in and out).

![CyberChef recipe: From Base64 then RC4 with passphrase TelemetryTea](Telemetry_stream_1.png)

### **Alternative Approaches (if applicable)**
I first tried Ghidra, since it's a reverse-engineering challenge, but I got lucky finding everything I needed in the `strings` output. If it hadn't been there, Ghidra would have been the way to recover the Base64 blob, the RC4 key and confirm the KSA bug.

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:* applied the recipe and hit **Bake!**, which recovered the note:

```
WEST PUMP STATION / NIGHT MAINTENANCE LOG

The collector archive was copied out of the engineering terminal after the night shift.
Most entries are ordinary valve checks and pressure readings, but one sentence was
marked by hand before the file was sealed.

If the recovered text is readable, the maintenance token is [FLAG REDACTED] and the checksum
label beside it can be ignored. The broken console on the floor is only a distraction.
```

The flag is the "maintenance token" in the recovered note — and the note itself tells me the checksum label next to it and the "broken console" (the binary's own broken output) are both red herrings.

## **Lessons Learned**
- Reverse engineering doesn't always mean opening Ghidra — `strings` alone can hand you the ciphertext, the key, and sometimes the whole solution.
- The description named the bug (the RC4 KSA starting from the wrong byte), so instead of fixing the binary I just ran a correct RC4 in CyberChef and ignored the program's output entirely.
- Watch for planted distractions: the "broken console" and the checksum label were both there to waste time.

## **References**
- <https://gchq.github.io/CyberChef/>
