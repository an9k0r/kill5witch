---
title: (BTLO/Investigation) - Total Recall
author:
  name: kill5witch
  link: https://github.com/an9k0r
date: 2022-05-22 17:33:00 +0100
categories: [BlueTeamLabs, Incident Response]
tags: [Redline]
math: true
mermaid: true
image:
  src: /assets/images/2022-05-22-21-47-48.png
  width: 694
  height: 515
  alt: image alternative text
---
**Microsoft Defender Antivirus has been there for over a decade now. It provides security against known threats until it has not been tampered with.**
- CTF is hosted on https://blueteamlabs.online/
# Scenario
> Unfortunately an Account Executive was just got let go from the company due to downsizing efforts. Before wiping his machine to be reissued in the future a threat hunter wants to conduct memory analysis after other suspicious activity was observed coming from the user in question before his termination was confirmed.
> Hopefully there’s nothing bad, but this could be an incident waiting to be discovered. See if you can find any malicious activity on their system. 

# Instructions
![](/assets/images/2022-05-20-21-28-17.png)

# Redline investigation
## Checking users
First of all let's try to get some basic info like existing users:
![](/assets/images/2022-05-20-22-42-31.png)

There are many users but apparently only `Jason` and `hacker` have been recently logged in. We should note us this two. Both are members of `Administrators` group and `Jason` is also member of `Remote Desktop Users`. Checking for RDP events we can also see that Firewall rule was changed
![](/assets/images/2022-05-22-19-49-24.png)

> What service was enabled by the attacker between 7-8 PM on 03/03/2021 which involves the netsh command and exception on the firewall? Provide the protocol and execution PID(5 points): RDP, 1028

Other users were created which can be seen in event logs searching for `4720` Event ID

![](/assets/images/2022-05-22-19-12-27.png)

> What user accounts were created by the insider? (alphabetical order)(5 points): Alice, Bob, Carlos, Jason, Smith

## Checking browser history
![](/assets/images/2022-05-22-19-19-17.png)
There are few interesting entries like, we get account which might be from attacker (or not, but it could be right?).

We also see that WinPwnage (https://github.com/rootm0s/WinPwnage) was visited.

Let's check downloads:

![](/assets/images/2022-05-22-19-27-35.png)

> The user tried to download an .exe file to the system but cancelled it. What was the filename?(5 points): Openvpn (1).exe

> What is the IP and port used to download other files to the system?(5 points): 10.201.1.105:8080

## Checking Tasks
Checking the Tasks, there are two interesting entries as it can be seen below.

![](/assets/images/2022-05-21-19-14-41.png)

> The task will grab the file from the attacker’s machine. What file is scheduled to run on the tasks? Include the full path to the file within the URL(5 points): http://10.201.1.105:8080/catchmeifyoucan.exe
> 
Both `powershell` Tasks have names as `DailyTask` and `OnIdleTask` and will be trigered at Logon or on `Idle` status.

> Can you find two scheduled tasks created by the malicious insider? (alphabetical order)(5 points): DailyTask, OnIdleTask

> What is the time creation for these scheduled tasks? (alphabetical order)(5 points): 2021-03-03 21:41:23Z, 2021-03-03 21:43:08Z

![](/assets/images/2022-05-21-19-16-35.png)

Task creation can be found in `Timeline`. I think it's the easiest way to search.

![](/assets/images/2022-05-21-19-39-01.png)

## Checking services
If we remove all Microsoft signed binaries we get short list of services.
![](/assets/images/2022-05-20-22-45-50.png)

Aparently there is Meterpreter service which is pretty much self-explainatory
![](/assets/images/2022-05-20-22-47-54.png)

We know the file name: `C:\Users\hacker\AppData\Local\Temp\OHpObWvB\metsvc.exe`. 

> Another common persistence method is using Windows Services. Identify any suspicious running services(5 points): C:\Users\hacker\AppData\Local\Temp\OHpObWvB\metsvc.exe, 31337

## Checking Registry
There's an entry in registry (checking manually), which runs  `UserInit.exe` and  `evil.exe` on logon.
![](/assets/images/2022-05-22-21-05-11.png)

> The insider has likely utilized ATT&CK ID T1547.001 to obtain persistence. Find the malicious registry entry and submit the Registry Key Value and modified date(5 points): Userinit.exe, evil.exe, 2021-03-07 11:15:29Z

> An attempt was made to reset a newly-created user's password some time after 9 PM on 03/03/2021. Find the Execution PID and Execution Thread ID(5 points): 472, 2872


