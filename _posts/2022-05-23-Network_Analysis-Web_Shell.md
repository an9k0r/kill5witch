---
title: [Investigation] - Network Analysis - Web Shell
author:
  name: kill5witch
  link: https://github.com/an9k0r
date: 2022-05-23 12:00:00 +0100
categories: [BlueTeamLabs, Incident Response]
tags: [Wireshare, TCPDump]
math: true
mermaid: true
image:
  src: /assets/images/2022-05-23-15-58-32.png
  width: 694
  height: 515
  alt: image alternative text
---
**The SOC received an alert in their SIEM for ‘Local to Local Port Scanning’ where an internal private IP began scanning another internal system.**
# Scenario
> The SOC received an alert in their SIEM for ‘Local to Local Port Scanning’ where an internal private IP began scanning another internal system.   

# Downloading the PCAP File
When downloading you should not extract the PCAP and replay it or something like that as PCAP contains real malware, at least that's what warning says on BTLO! :)

Anyways let's get going.

# Finding PortScan
After opening the PCAP and checking Statistics, it's obvious that there is one IP that appears to be running port scan.
![](/assets/images/2022-05-23-15-27-03.png)

Packets are almost the same lenght and destination ports are random and in `lower` range. We can sort on low/high ports just by clicking on `Port B` in my case. If there were other IPs involved, we might need to filter on IP first.

> What is the IP responsible for conducting the port scan activity? (1 points): 10.251.96.4

> What is the port range scanned by the suspicious host? (1 points): 1-1024

## Diving deeper
In order what kind of scan it is, we can follow TCP flow and as it appears it's TCP with just `SYN` flag set.

![](/assets/images/2022-05-23-15-34-36.png)

> What is the type of port scan conducted? (1 points): TCP SYN

To actually get which ports are open, we can assume that more packets were sent between victim and the client. Just by checking Statistics, this appear to be the 22, 80. Perhaps worth mentioning, there was communication to 4422 (from victim to attackers machine!)

![](/assets/images/2022-05-23-15-38-50.png)

## Finding TTPs
As many tools use specific UserAgent Headers, they will stand out if we focus on them. To get `User Agent` as a column, just right-click it on any http packet and choose `Apply as a Column`. Beforehand i filtered on TCP.PORT, IP.SRC and HTTP 
```
(tcp.port==80 && http) && (ip.src == 10.251.96.4)
```

![](/assets/images/2022-05-23-15-45-00.png)

> Two more tools were used to perform reconnaissance against open ports, what were they? (1 points): gobuster 3.0.1, sqlmap 1.4.7

## Initial Foothold analysis
Attacker used `editprofile.php` which most probably triggered form `upload.php`. Actual commands were executed in `/uploads/dbfunctions.php`

![](/assets/images/2022-05-23-15-52-07.png)

> What is the name of the php file through which the attacker uploaded a web shell? (1 points): editprofile.php

> What is the name of the web shell that the attacker uploaded? (1 points): dbfunctions.php

> What is the parameter used in the web shell for executing commands? (1 points): cmd

> What is the first command executed by the attacker? (1 points): id

> What is the type of shell connection the attacker obtains through command execution? (1 points): reverse shell

> What is the port he uses for the shell connection? (1 points): 4422

Checking the packet 16102 which sent `upload.php` we can see what uploaded file was:

![](/assets/images/2022-05-23-15-54-37.png)