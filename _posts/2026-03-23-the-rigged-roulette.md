---
title: "UVT CTF – The Rigged Roulette"
date: 2026-03-23 08:00:00 +0200
categories: [CTF, UVT CTF]
tags: [lcg, linear-congruential-generator, prng, modular-arithmetic, modular-inverse, pwntools, known-plaintext, crypto-math]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  The Rigged Roulette  
**Category**:  cryptography  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    easy  
**Tags:**  LCG, linear-congruential-generator, PRNG, modular-arithmetic, modular-inverse, pwntools, known-plaintext, crypto-math  
**Date solved:**    23/03/2026 (DD/MM/YYYY)  
**Challenge Description**:  Welcome to the Rigged Roulette Casino! We guarantee our random number generator is fully "secure". It's built on pure math, so it MUST be safe. We're so confident in our impenetrable system that we offer a $1,000,000 jackpot to anyone who can guess the next winning number! Surely nobody could exploit our proprietary roulette wheel and steal the Grand Prize... right? Are you ready to test your luck?  
**Files provided**:  source code  

## **Environment and Tools** 
**Operating System**:  Windows, Kali Linux  
**Tools/ Software**:  Visual Studio Code, Google, ChatGPT  
**Programming Languages (if any)**:  python (script)  

## **Recognition Pattern**
- A "secure" random number generator in sarcastic quotes → the RNG is weak and predictable
- The source used an LCG (`state = (a * state + c) % M`) → breakable, I can recover `a` and `c` from 3 consecutive outputs
- I could "spectate" and watch numbers before betting → collect samples, do the math, predict the next one
- The modulus was a Mersenne prime → the modular inverse is clean since `M` is prime

## **Approach and Methodology**
### **Initial Analysis**
When I first opened the challenge, I saw I'd have to connect to a server, and from my previous CTFs for this class I assumed I'd need a Python script. I opened the server code to understand what it does and what my script had to do. After a quick read, I realized I'd have to play roulette to win the flag — which meant guessing the generated number.  

### **Steps Taken**
Reading further, I found how the numbers were generated:

```python
M = 2**61 - 1  # 2305843009213693951, a Mersenne prime

class LCGCasino:
    def __init__(self):
        # The multiplier and increment are secret!
        self.a = random.randint(2, M - 1)
        self.c = random.randint(2, M - 1)
        self.state = random.randint(2, M - 1)
    ...
    def next_number(self):
        self.state = (self.a * self.state + self.c) % M
        return self.state
```

There were two secret values to "guess" (`a` and `c`) in order to predict every number that followed.  

I ran the server code locally to confirm what it was asking: I was supposed to play, determine `a` and `c`, then bet on the next number and win enough money to buy the flag.  

The final step was to write a Python script to do the math and interact with the server automatically until it was time to buy the flag. This part was slightly tricky because I needed `pwn` and `inverse` from `Crypto.Util.number`. I used ChatGPT for documentation help since it was faster than searching online myself.  

The final script connects, reads output until the server asks for an option, spectates 3 times, recovers `a` and `c`, calculates the next number, and bets on it — winning the money. The rest of the interaction is left to the user.

```python
from pwn import *
from Crypto.Util.number import inverse

context.log_level = 'debug'  # shows full interaction

HOST = "194.102.62.166"
PORT = 28253
M = 2**61 - 1

def get_number(line):
    return int(line.split(": ")[1])

r = remote(HOST, PORT)

# read until first menu
r.recvuntil(b"Your choice:")

nums = []
# spectate 3 times
for _ in range(3):
    r.sendline(b"1")
    while True:
        line = r.recvline()
        if b"landed on" in line:
            num = get_number(line.decode())
            print("Extracted:", num)
            nums.append(num)
            break
    r.recvuntil(b"Your choice:")

x0, x1, x2 = nums

# recover a
a = ((x2 - x1) * inverse(x1 - x0, M)) % M
# recover c
c = (x1 - a * x0) % M
print(f"[+] a = {a}")
print(f"[+] c = {c}")

# predict next
next_val = (a * x2 + c) % M
print(f"[+] Next number = {next_val}")

# place bet
r.sendline(b"2")
r.recvuntil(b"Enter your guess")
r.sendline(str(next_val).encode())

# print everything after
r.interactive()
```

All that was left was to run the script and watch it work.  

![the script spectating and recovering a and c](Rigged_Roulette_1.png)

![the script predicting the next number and betting](Rigged_Roulette_2.png)

### **Alternative Approaches (if applicable)**
Another method would have been doing the math by hand — there were only a few calculations. But the huge numbers and the need for efficiency made me drop that idea almost instantly.  

*Update:*  
I did this ctf again some time later and came up with a different script than last time, same result but a simpler approach. No need to interact with the terminal, the script does all the work.    

```python
from pwn import *


def inverse(a,m):
    return pow(a, -1, m)

# nc 194.102.62.183 22497
conn=remote('194.102.62.183',22497)

M = 2**61 - 1 

spins=0
spenRez=[]

while spins <3:
    print(conn.recvuntil(b'Your choice: '))
    conn.sendline(str(1).encode())

    print(conn.recvuntil(b'[Spin complete] The wheel landed on: '))
    spenRez.append(int(conn.recvline().strip()))
    spins=spins+1


a=((spenRez[2]-spenRez[1])* inverse(spenRez[1]-spenRez[0], M)) %M
c=(spenRez[1]- a*spenRez[0]) %M

nextNo= (a*spenRez[2]+c)%M

print(conn.recvuntil(b'Your choice: '))
conn.sendline(str(2).encode())

print(conn.recvuntil(b'Enter your guess for the next number: '))
conn.sendline(str(nextNo).encode())


print(conn.recvuntil(b'Your choice: '))
conn.sendline(str(3).encode())

# important so that we actually recieve the very last byte
print(conn.recvall().decode(errors="ignore"))
conn.close()
```

## **Solution and Flag Retrieval**
After winning enough money, all that was left was to actually purchase the flag and stop the script.  

![buying the flag after winning](Rigged_Roulette_3.png)

*Final Exploit / Command Used:* menu option `3` (Buy the Flag), after the script recovered `a`/`c` and won the jackpot.  

## **Lessons Learned**
- The core lesson: an LCG isn't secure. With 3 consecutive outputs (`x0`, `x1`, `x2`) I recovered the hidden parameters — `a = (x2 - x1) * inverse(x1 - x0, M) % M` and `c = (x1 - a * x0) % M`.
- `inverse()` from `Crypto.Util.number` computes the modular inverse, which the recovery needs. It works here because `M` is prime.
- pwntools did the server interaction: `remote(HOST, PORT)` to connect, `recvuntil()` to wait for prompts, `sendline()` to bet, `interactive()` to take over at the end.
- When a challenge lets you spectate before acting, grab as many samples as you can first — those observations were the whole attack here, and I only needed 3.

## **References**
- <https://chatgpt.com/>
