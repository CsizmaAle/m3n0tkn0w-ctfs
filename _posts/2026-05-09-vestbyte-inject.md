---
title: "UVT CTF – VestByte Inject"
date: 2026-05-09 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [web, command-injection, os-injection, shell-injection, grep, bash, metacharacters, shell-true]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  VestByte Inject  
**Category**:  web exploitation  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    easy  
**Tags:**  web, command-injection, OS-injection, shell-injection, grep, bash, metacharacters, shell-true  
**Date solved:**    9/5/2026 (DD/MM/YYYY)  
**Challenge Description**:  VestByte Institute runs an internal archive portal where students open course notes from local files. Your objective is to inspect the request flow, understand how the file path is built, and recover the flag from the server (you have to search and find the flag with commands).  
**Files provided**:  server connection  

## **Environment and Tools** 
**Operating System**:  Windows  
**Tools/ Software**:  Google  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- I put junk in the search field and the page handed back a shell command name (`grep`) — that's my tell it's OS command injection, not SQL or XSS
- The challenge is about searching through local files ("archive portal", "course notes from local files"), which points at a shell command rather than a database query
- If the backend is careless enough to leak `grep` at me, it's usually careless enough to let `;` or `|` chain my own commands onto it

## **Approach and Methodology**
### **Initial Analysis**
I knew there was some injection involved based on the description, and after opening the web page and seeing the search button it was clear that was the entry point — I just didn't know which type of injection yet.  

### **Steps Taken**
I first typed some gibberish in the search bar to see what would happen, and saw something interesting: `grep`.  

![search output leaking a grep command](VestByte_Inject_1.png)

So it was using bash commands, which meant it was easy to exploit. I went with `ls` and `cat` and looked through the files and folders that popped up (e.g. `; ls; ls templates; cat templates/learn.html`).  

In `templates/learn.html` I found an interesting piece of information:  

```html
<h2>OS Command Injection in a Search Helper</h2>
<p>
Command injection happens when user input is inserted into a shell command without proper separation or escaping.
In this challenge, the archive search helper takes a user-supplied search string and inserts it directly into
a shell pipeline that runs <code>cat</code> and <code>grep</code>.
</p>
<p>
Because the backend uses <code>shell=True</code>, metacharacters such as <code>;</code>, <code>&amp;&amp;</code>,
or <code>|</code> let an attacker append new commands. A payload like
<code>graph; cat /flag.txt #</code> causes the archive helper to print both grep output and the flag.
</p>
```

So I ran that command: `graph; cat /flag.txt`.  

### **Alternative Approaches (if applicable)**
Another type of injection I tried but failed with was SQL injection, and I was also considering XSS — but the output area returning the grep info made me realize it was bash.  

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:*
```
graph; cat /flag.txt
```

## **Lessons Learned**
- The biggest thing I took away: when I suspect injection but don't know the type, I let the server tell me. Shell command names coming back means OS command injection. I'd been half-guessing between SQLi and XSS until the `grep` output made it obvious.
- `;` is the simplest separator — `anything; my_command` just runs both. Good to know the others exist too (`&&`, `||`, `|`), but `;` was all I needed here.
- Once I was in, the moves that mattered were `; ls` to look around and `; cat` to read files, which is how I found the hint in `templates/learn.html`.
- I also learned a `#` at the end comments out whatever the server tacks on after my input, so the command doesn't error out — the intended payload was `graph; cat /flag.txt #`.


## **References**
none used
