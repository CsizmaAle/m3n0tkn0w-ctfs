---
title: "RSTCON 2026 – Gauge Height"
date: 2026-10-25 08:09:00 +0200
categories: [CTF, RSTCON 2026]
tags: [osint, geolocation, usgs, scada, csv, google-maps]
media_subpath: /assets/img/posts/
---

## **Challenge Information**
**Title:**  Gauge Height  
**Category**:  OSINT  
**Platform:**  RSTCON  
**Difficulty:**  easy  
**Tags:**  OSINT, geolocation, USGS, SCADA, csv, google-maps  
**Date solved:**  25/10/2026 (DD/MM/YYYY)  
**Challenge Description**:  We pulled a historian export off a hydro SCADA box. The operator stripped the site name and coordinates, but they left the upstream data feed's identifiers in the header. Find out exactly what this gauge is sitting below.  
500m Radius Tolerance  
**Files provided**:  historian_export.csv  

## **Environment and Tools** 
**Operating System**:  Kali Linux (Parallels)  
**Tools/ Software**:  LibreOffice, Mozilla, Google Maps  
**Programming Languages (if any)**:  bash  

## **Recognition Pattern**
- A data export with an agency name and a site ID in the header => a public monitoring station that can be looked up
- "500m Radius Tolerance" => the answer is a set of coordinates

## **Approach and Methodology**
To solve this challenge i first looked at the csv and there was some useful information. 

![the csv header with the USGS agency and site ID](Gauge_Height_1.png)

The agency USGS was important and the site=09421500. 

So i went on Mozilla and looked for the agency's website: https://www.usgs.gov/

Once on the website i searched up the site and an article popped up, regarding a survey at a bridge.  
![USGS search result for site 09421500](Gauge_Height_2.png)

i looked up the bridge on google maps for the coordinates, as they were asked.  
![the gauge location on Google Maps, below Hoover Dam](Gauge_Height_3.png)

Since the gage_height was high, i decided to put the pin slightly north-east of the gage. 
## **Solution and Flag Retrieval**
Final Exploit / Command Used: Copied the coordinates  
Coordinates: [FLAG REDACTED]

## **Lessons Learned**
- a site ID from a data feed can be searched on the agency's website (here USGS) to find the exact location
- USGS parameter codes: 00060 = streamflow, 00065 = gage height

## **References**
- USGS (https://www.usgs.gov/)
- Google Maps (https://www.google.com/maps)
