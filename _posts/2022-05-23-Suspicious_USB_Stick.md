---
title: [Investigation] - Suspicious USB Stick
author:
  name: kill5witch
  link: https://github.com/an9k0r
date: 2022-05-23 12:00:00 +0100
categories: [BlueTeamLabs, Digital Forensics]
tags: [XXD, strings, VirusTotal, yara]
math: true
mermaid: true
image:
  src: /assets/images/2022-05-23-19-44-49.png
  width: 694
  height: 515
  alt: image alternative text
---
**One of our clients informed us they recently suffered an employee data breach...can you check the contents on the USB drive?**
# Scenario
> One of our clients informed us they recently suffered an employee data breach. As a startup company, they had a constrained budget allocated for security and employee training. I visited them and spoke with the relevant stakeholders. I also collected some suspicious emails and a USB drive an employee found on their premises. While I am analyzing the suspicious emails, can you check the contents on the USB drive?   

Unzip the contents. I'll be using SIFT Workstation from SANS: https://www.sans.org/tools/sift-workstation/

```bash
sansforensics@siftworkstation: ~/Desktop/cases_CTF/BTLO Suspicious USB
$ unzip USB.zip 
Archive:  USB.zip
   creating: USB/
   creating: USB/autorun/
[USB.zip] USB/autorun/autorun.inf password: 
 extracting: USB/autorun/autorun.inf  
  inflating: USB/autorun/README.pdf  
```

# Digital Forensics - the Beginning 

Autorun.inf runs `README.pdf` which obviously isn't nice thing to do.
```
sansforensics@siftworkstation: ~/Desktop/cases_CTF/BTLO Suspicious USB/USB
$ cat autorun/autorun.inf 
[autorun]
open=README.pdf
icon=autorun.ico
```
More info here:https://en.wikipedia.org/wiki/Autorun.inf 
In short, `Autorun.inf` autoruns components on Windows OS.

> What file is the autorun.inf running? (3 points): README.pdf

## Upload README.pdf to VirusTotal

Uploading file to VirusTotal shows that file is most likely malicious:
(I had to use version for old-browsers, but it's still VirusTotal)
![](/assets/images/2022-05-23-19-28-50.png)

> Does the pdf file pass virustotal scan? (No malicious results returned) (2 points): False

## Checking Magic Bytes
Comparing magic bytes reveals that file is indeed a `PDF` as it claims to be

![](/assets/images/2022-05-23-19-31-15.png)

![](/assets/images/2022-05-23-19-32-07.png)
Link: https://en.wikipedia.org/wiki/List_of_file_signatures

> What OS type can the file exploit? (Linux, MacOS, Windows, etc) (5 points): Windows

## Checking for command Execution in PDF
Simple search for exe reveals that PDF should execute `cmd.exe`

![](/assets/images/2022-05-23-19-34-12.png)

> A Windows executable is mentioned in the pdf file, what is it? (3 points): `cmd.exe`

PDF contains one `OpenAction`.
![](/assets/images/2022-05-23-19-41-10.png)

> How many suspicious /OpenAction elements does the file have? (5 points): `1`

# Further Analysis
## Yara Scan with Maldoc_PDF.yar
```
sansforensics@siftworkstation: ~/Desktop/cases_CTF/BTLO Suspicious USB/USB/autorun
$ yara -w /home/sansforensics/tools/rules/maldocs/Maldoc_PDF.yar README.pdf 
suspicious_launch_action README.pdf
suspicious_embed README.pdf
multiple_versions README.pdf
PDF_Embedded_Exe README.pdf
```

