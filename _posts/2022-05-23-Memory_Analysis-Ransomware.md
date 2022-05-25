---
title: (Challenge) - Memory Analysis - Ransomware
author:
  name: kill5witch
  link: https://github.com/an9k0r
date: 2022-05-23 12:00:00 +0100
categories: [BlueTeamLabs, Digital Forensics]
tags: [Volatility, Ransomware, WannaCry]
math: true
mermaid: true
image:
  src: /assets/images/2022-05-23-19-03-26.png
  width: 694
  height: 515
  alt: image alternative text
---
**The Account Executive called the SOC earlier and sounds very frustrated and angry. He stated he can’t access any files on his computer and keeps receiving a pop-up stating that his files have been encrypted.**
# Scenario
> The Account Executive called the SOC earlier and sounds very frustrated and angry. He stated he can’t access any files on his computer and keeps receiving a pop-up stating that his files have been encrypted. You disconnected the computer from the network and extracted the memory dump of his machine and started analyzing it with Volatility. Continue your investigation to uncover how the ransomware works and how to stop it!    

I'll be using SIFT Workastation from SANS. https://www.sans.org/tools/sift-workstation/

# Forensics - The Begining 
At first, we get a `.mem` file which we can easily read with `volatility` and this is also what we're expected to do.

![](/assets/images/2022-05-23-18-21-47.png)

> Run “vol.py -f infected.vmem --profile=Win7SP1x86 psscan” that will list all processes. What is the name of the suspicious process? (3 points): `@WanaDecryptor`

`@WanaDecryptor` definately sticks out here, along with `or4qtckT.exe`. Checking PPID `2732` of `@WanaDecryptor`. 

Googling the Filenames reveals what the Ransoware should be
> Can you identify what ransomware it is? (Do your research!) (2 points): WannaCry

Checking the Process Tree it becomes more obvious

![](/assets/images/2022-05-23-18-28-11.png)

> What is the parent process ID for the suspicious process? (3 points): 2732

> What is the initial malicious executable that created this process? (3 points): `or4qtckT.exe`

We can also get graphical view of process tree including exited processes
```bash
sansforensics@siftworkstation: ~/Desktop/cases_CTF/BTLO Memory Analysis - Ransomware
$ vol.py -f infected.vmem --profile=Win7SP1x86 psscan --output dot --output-file psscan.dot
Volatility Foundation Volatility Framework 2.6
Outputting to: psscan.dot

sansforensics@siftworkstation: ~/Desktop/cases_CTF/BTLO Memory Analysis - Ransomware
$ dot -Tpng psscan.dot -o psscan.png

![](/assets/images/2022-05-23-18-35-30.png)

> If you drill down on the suspicious PID (vol.py -f infected.vmem --profile=Win7SP1x86 psscan | grep (PIDhere)), find the process used to delete files (3 points): `taskdl.exe`
```

# Taking deeper look - handles / dlllist
Using `dlllist` module with the process should reveal loaded libraries and `.exe0`

![](/assets/images/2022-05-23-18-47-30.png)

> Find the path where the malicious file was first executed (3 points): `C:\Users\hacker\Desktop\or4qtckT.exe`

Checking handles reveals the `.eky` file which is encrypted private key with embeded public key.

![](/assets/images/2022-05-23-18-50-36.png)

> What is the filename for the file with the ransomware public key that was used to encrypt the private key? (.eky extension) (3 points): `00000000.eky`

Cmdline

![](/assets/images/2022-05-23-18-58-59.png)

Thanks for reading!

# Resource
https://resources.infosecinstitute.com/topic/ransomware-analysis-with-volatility/

https://www.sans.org/tools/sift-workstation/