---
title: "WolvCTF 2026 – In Jeopardy!"
date: 2026-11-21 14:00:00 +0200
categories: [Events, WolvCTF 2026]
tags: [reverse-engineering, ghidra, cpp, tar, gzip, caesar, shift-by-3, python]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  In Jeopardy!  
**Category**:  reverse engineering  
**Platform:**  WolvCTF  
**Difficulty:**  
**Tags:**  reverse-engineering, Ghidra, C++, tar, gzip, caesar, shift-by-3, python  
**Date solved:**  21/11/2026 (DD/MM/YYYY)  
**Challenge Description**:  Welcome to Jeopardy! Run the executable to see what your first question is. Don't know the answer? Then you might have some digging to do...  
**Files provided**:  dis.tar.gz  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  Ghidra  
**Programming Languages (if any)**:  bash, python  

## **Recognition Pattern**
- A binary that asks for a password and runs some operation on hardcoded data => read the data and undo the operation
- The "Correct!" path runs print_key_message, which transforms a hardcoded string => that transform is what recovers the flag
- C++ decompiles into a lot of noise in Ghidra; the real logic is just a few lines inside the loop

## **Approach and Methodology**
### **Initial Analysis**
After reading the challenge description it was obvious there is something hardcoded within the binary, and some operations are done on it. 

### **Steps Taken**

- unarchive the .gz layer using gzip -d file
- unarchive the .tar layer using tar -xvf file

=> obtain a file in-jeopardy  
checked the file with file and confirmed it was an executable.  
Strings show that it is a program that requires the user to input a password. 
```bash
zfwi~Fk744Hqj680i3UbwZ30KXQGU6G
Correct! 
This password is used to gain the flag from the user in the Jeopardy Rev challenge.
What is 
Incorrect. Try starting your statement with What is
Incorrect.
```

i decided to open ghidra and try to analyze the exe.

I found some interesting code: 
```C

long ydhms_diff(ulong param_1,long param_2,int param_3,int param_4,int param_5,uint param_6,
               int param_7,int param_8,int param_9,int param_10)

{
  int iVar1;
  int iVar2;
  int iVar3;
  int iVar4;
  
  iVar2 = ((int)((long)param_1 >> 2) + 0x1db) - (uint)((param_1 & 3) == 0);
  iVar4 = ((int)((long)(int)param_6 >> 2) + 0x1db) - (uint)((param_6 & 3) == 0);
  iVar1 = iVar2 / 0x19 + (iVar2 % 0x19 >> 0x1f);
  iVar3 = iVar4 / 0x19 + (iVar4 % 0x19 >> 0x1f);
  return ((long)param_5 +
         (((long)param_4 +
          (((long)param_3 +
           (((param_2 + (param_1 - (long)(int)param_6) * 0x16d) - (long)param_7) +
           (long)(((iVar1 >> 2) - (iVar3 >> 2)) + ((iVar2 - iVar4) - (iVar1 - iVar3)))) * 0x18) -
          (long)param_8) * 0x3c) - (long)param_9) * 0x3c) - (long)param_10;
}






/* WARNING: Unknown calling convention -- yet parameter storage is locked */
/* print_key_message() */

void print_key_message(void)

{
  bool bVar1;
  char *pcVar2;
  ostream *poVar3;
  undefined8 local_78;
  undefined8 local_70;
  string local_68 [32];
  string local_48 [37];
  allocator local_23;
  allocator local_22;
  char local_21;
  string *local_20;
  
  std::allocator<char>::allocator();
                    /* try { // try from 00404b63 to 00404b67 has its CatchHandler @ 00404c6d */
  std::__cxx11::string::string<>(local_48,&DAT_0055a008,&local_23);
  std::allocator<char>::~allocator((allocator<char> *)&local_23);
  std::allocator<char>::allocator();
                    /* try { // try from 00404b90 to 00404b94 has its CatchHandler @ 00404c87 */
  std::__cxx11::string::string<>(local_68,"",&local_22);
  std::allocator<char>::~allocator((allocator<char> *)&local_22);
  local_20 = local_48;
  local_70 = std::__cxx11::string::begin(local_20);
  local_78 = std::__cxx11::string::end(local_20);
  while( true ) {
    bVar1 = __gnu_cxx::operator!=((__normal_iterator *)&local_70,(__normal_iterator *)&local_78);
    if (!bVar1) break;
    pcVar2 = (char *)__gnu_cxx::__normal_iterator<>::operator*((__normal_iterator<> *)&local_70);
    local_21 = *pcVar2;
                    /* try { // try from 00404bf0 to 00404c52 has its CatchHandler @ 00404c98 */
    std::__cxx11::string::operator+=(local_68,local_21 + -3);
    __gnu_cxx::__normal_iterator<>::operator++((__normal_iterator<> *)&local_70);
  }
  poVar3 = std::operator<<((ostream *)&std::cout,"Correct! ");
  poVar3 = (ostream *)std::ostream::operator<<(poVar3,std::endl<>);
  poVar3 = std::operator<<(poVar3,local_68);
  std::ostream::operator<<(poVar3,std::endl<>);
  std::__cxx11::string::~string(local_68);
  std::__cxx11::string::~string(local_48);
  return;
}



```

The main function looked slightly different from what i'm used to.
```c

undefined8 main(void)

{
  char cVar1;
  bool bVar2;
  ostream *poVar3;
  long lVar4;
  istream *piVar5;
  string local_78 [32];
  string local_58 [32];
  string local_38 [40];
  
  poVar3 = std::operator<<((ostream *)&std::cout,
                           "This password is used to gain the flag from the user in the Jeopardy Rev challenge."
                          );
  std::ostream::operator<<(poVar3,std::endl<>);
  std::__cxx11::string::string(local_78);
  do {
    piVar5 = std::getline<>((istream *)&std::cin,local_78);
    bVar2 = std::ios::operator.cast.to.bool((ios *)(piVar5 + *(long *)(*(long *)piVar5 + -0x18)));
    if (!bVar2) {
LAB_00404df0:
      std::__cxx11::string::~string(local_78);
      return 0;
    }
                    /* try { // try from 00404d01 to 00404d05 has its CatchHandler @ 00404e16 */
    key[abi:cxx11]();
                    /* try { // try from 00404d16 to 00404d1a has its CatchHandler @ 00404e05 */
    std::operator+((char *)local_58,(string *)"What is ");
    cVar1 = std::operator==(local_78,local_58);
    std::__cxx11::string::~string(local_58);
    std::__cxx11::string::~string(local_38);
    if (cVar1 != '\0') {
                    /* try { // try from 00404d4c to 00404de7 has its CatchHandler @ 00404e16 */
      print_key_message();
      std::operator>>((istream *)&std::cin,local_78);
      goto LAB_00404df0;
    }
    lVar4 = std::__cxx11::string::find(local_78,"What is ",0);
    if (lVar4 == -1) {
      poVar3 = std::operator<<((ostream *)&std::cout,
                               "Incorrect. Try starting your statement with What is");
      std::ostream::operator<<(poVar3,std::endl<>);
    }
    else {
      poVar3 = std::operator<<((ostream *)&std::cout,"Incorrect.");
      std::ostream::operator<<(poVar3,std::endl<>);
    }
  } while( true );
}


```

The print_key_message is the function that i need to rev in order to get the flag. The data is &DAT_0055a008, so i copied it as a python list and asked claude to help figure out the code.

Apparently most of the code is actually noise, and all i need to look at are a few lines of code:
```C
  while( true ) {
    bVar1 = __gnu_cxx::operator!=((__normal_iterator *)&local_70,(__normal_iterator *)&local_78);
    if (!bVar1) break;
    pcVar2 = (char *)__gnu_cxx::__normal_iterator<>::operator*((__normal_iterator<> *)&local_70);
    local_21 = *pcVar2;
    std::__cxx11::string::operator+=(local_68,local_21 + -3);
    __gnu_cxx::__normal_iterator<>::operator++((__normal_iterator<> *)&local_70);
  }
```

what it does, it subtracts 3 from each character. 

## **Solution and Flag Retrieval**
Final Exploit / Command Used: ./solver.py 
```python3
#!/usr/bin/python3

data =[ 0x7a, 0x66, 0x77, 0x69, 0x7e, 0x46, 0x6b, 0x37, 0x34, 0x34, 0x48, 0x71, 0x6a, 0x36, 0x38, 0x30, 0x69, 0x33, 0x55, 0x62, 0x77, 0x5a, 0x33, 0x30, 0x4b, 0x58, 0x51, 0x47, 0x55, 0x36, 0x47, 0x80 ]

text=""

for c in data:
	text+=chr((c-3)& 0xff)

print(text)
```

## **Lessons Learned**
- C++ code is very noisy in ghidra and a lot of lines of code are pointless
- find the function on the "Correct" path and read only the transform loop inside it

## **References**
- claude.ai (https://claude.ai)
