---
title: "UVT CTF – VestByte Templates"
date: 2026-05-09 08:01:00 +0200
categories: [Platform, UVT CTF]
tags: [web, ssti, jinja2, template-injection, environment-variables, code-execution]
---

{% raw %}
## **Challenge Information**
**Title:**  VestByte Templates  
**Category**:  web exploitation  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    easy  
**Tags:**  web, SSTI, Jinja2, template-injection, environment-variables, code-execution  
**Date solved:**    9/5/2026 (DD/MM/YYYY)  
**Challenge Description**:  VestByte Institute also offers a student badge generator that builds a personalized welcome card from user input. The application renders the submitted name in a way that is more powerful than it should be. Your mission is to exfiltrate the flag from the environment variables, or just use code execution to find it.  
**Files provided**:  connection to the web page  

## **Environment and Tools** 
**Operating System**:  Linux (Ubuntu)  
**Tools/ Software**:  Mozilla, VirtualBox  
**Programming Languages (if any)**:  Jinja2  

## **Recognition Pattern**
- The description saying it renders the name "in a way more powerful than it should be" → that's pointing straight at template injection
- User input landing in a generated page (here a badge/welcome card) → worth testing for SSTI
- I tested each injection type with something simple; `{{7*7}}` coming back as `49` is what confirmed Jinja2 SSTI
- "Exfiltrate from environment variables" → the flag's in an env var, so I need SSTI to reach `os.environ`

## **Approach and Methodology**
### **Initial Analysis**
From the challenge description I got that the way the name is handled is inappropriate, meaning it's open to injection. At first I had to consider all the possible injections: XSS, bash, SQL and Jinja2.  

### **Steps Taken**
I tested each type of injection with something simple to see how it reacted. When I ran `{{7*7}}` and the output was `49`, it was clear the injection was Jinja2. I came back to this playbook to see what I used last time and ran the same command to get the flag.  

### **Alternative Approaches (if applicable)**
Instead of entering `{{ lipsum.__globals__["o"~"s"].environ.get("FLAG") }}`, the alternative is `{{ lipsum.__globals__["o"~"s"].environ }}`, which returns all the environment variables together with the flag.  

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:*
```
{{ lipsum.__globals__["o"~"s"].environ.get("FLAG") }}
```

## **Lessons Learned**
- Best habit here: test the injection type before committing to an attack. I ran a few simple things and `{{7*7}}` → `49` told me it was Jinja2 straight away.
- This was the same SSTI as meme.social but with no blocklist, so I didn't even need the `"o"~"s"` trick — plain `["os"]` would've worked. What actually saved me time was coming back to this playbook, looking up what I used last time, and reusing it. That's exactly why I document these.
- `{{ lipsum.__globals__["o"~"s"].environ }}` dumps every env var at once, which is handy when I don't know the exact variable name.

## **References**
none used

{% endraw %}
