---
title: (BTLO/Challenge) - The Planet's Prestige / Email Analysis
author:
  name: kill5witch
  link: https://github.com/an9k0r
date: 2022-05-25 05:00:00 +0100
categories: [BlueTeamLabs, CTF-Like]
tags: [Email Analysis, exiftool, XML-Parsing]
math: true
mermaid: true
image:
  src: /assets/images/2022-05-25-11-25-12.png
  width: 694
  height: 515
  alt: image alternative text
---
**It's all about an Email Analysis with small CTF-y part**
- CTF is hosted on https://blueteamlabs.online/
# Scenario
> CoCanDa, a planet known as 'The Heaven of the Universe' has been having a bad year. A series of riots have taken place across the planet due to the frequent abduction of citizens, known as CoCanDians, by a mysterious force. CoCanDa’s Planetary President arranged a war-room with the best brains and military leaders to work on a solution. After the meeting concluded the President was informed his daughter had disappeared. CoCanDa agents spread across multiple planets were working day and night to locate her. Two days later and there’s no update on the situation, no demand for ransom, not even a single clue regarding the whereabouts of the missing people. On the third day a CoCanDa representative, an Army Major on Earth, received an email.

# Intro
For this investigation, i'll follow the questions sprovided by BTLO.

I'll be using Sublime Text in Sift WorkstationVM with Email Header (https://packagecontrol.io/packages/Email%20Header) plugin, to get some highlighting on headers.

There are many header present in Email messages being sent. Some are standardized (https://www.iana.org/assignments/message-headers/message-headers.xhtml) and some are experemental or extended and they have `X-` prepended.

Now let's begin.

# The Beginning
The topmost `Recieved` Header is the one closest to destination, where the lowest one is closer to the sender. Apparently client was using `postfix` which is why we nee to look in the second `Recieved` header in the middle on line 28. 
![](/assets/images/2022-05-24-22-25-46.png)

> What is the email service used by the malicious actor? (1 points): `emkei.cz`

# THE interessting headers
Let's take the look to different headers for the sake of learning

## Reply-To
> When the "Reply-To:" field is present, it indicates the mailbox(es) to which the author of the message suggests that replies be sent. Link: https://datatracker.ietf.org/doc/html/rfc4021#page-8

![](/assets/images/2022-05-24-22-28-06.png)

> What is the Reply-To email address? (2 points): negeja3921@pashter.com

## Errors-To
> A bounce address is an email address to which bounce messages are delivered. There are many variants of the name, none of them used universally, including return path, reverse path, envelope from, envelope sender, MAIL FROM, 5321-FROM, return address, From_, **Errors-to**, etc. It is not uncommon for a single document to use several of these names.  Link: https://en.wikipedia.org/wiki/Bounce_address

## Return-Path
> Return path for message response diagnostics. Link: https://datatracker.ietf.org/doc/html/rfc4021#page-17

## Received-SPF: fail and Authentication-Results
We can that `Recieved-SPF` returns a fail
![](/assets/images/2022-05-25-06-57-57.png)

`Google.com` has a DNS TXT record set with SPF information in it which basically states what is allowed and what not.

### Digging deeper in SPF
So let's manually check the SPF policy from `google.com`.
```
sansforensics@siftworkstation: ~/Desktop/cases_CTF
$ dig google.com txt | grep spf
google.com.		5	IN	TXT	"v=spf1 include:_spf.google.com ~all"
```
First DNS TXT record of `google.com` says we need to check `_spf.google.com` so let's do that.
```
sansforensics@siftworkstation: ~/Desktop/cases_CTF
$ dig _spf.google.com txt | grep spf
; <<>> DiG 9.10.3-P4-Ubuntu <<>> _spf.google.com txt
;_spf.google.com.		IN	TXT
_spf.google.com.	5	IN	TXT	"v=spf1 include:_netblocks.google.com include:_netblocks2.google.com include:_netblocks3.google.com ~all"
```
Now we get more information where the Emails from `google.com` should actually come from.
- `v=spf1` tells us a version
- `include:_netblocks.google.com include:_netblocks2.google.com include:_netblocks3.google.com` is the actual policy. We can resolve the `_netblocks` simply by asking DNS
```
sansforensics@siftworkstation: ~/Desktop/cases_CTF
$ dig _netblocks.google.com txt | grep spf
_netblocks.google.com.	5	IN	TXT	"v=spf1 ip4:35.190.247.0/24 ip4:64.233.160.0/19 ip4:66.102.0.0/20 ip4:66.249.80.0/20 ip4:72.14.192.0/18 ip4:74.125.0.0/16 ip4:108.177.8.0/21 ip4:173.194.0.0/16 ip4:209.85.128.0/17 ip4:216.58.192.0/19 ip4:216.239.32.0/19 ~all"
```
- `~all` means that all other senders should be disallowed. 

If we check the senders Email, we'd see that it does not come from allowed IP range, hence `fail`

# Attachments
Attachments are attached to the Emails like documents are send over HTTP POST Requests. 
![](/assets/images/2022-05-25-07-17-53.png)

Filename actually says it's PDF but if we copy whole Base64, decode it and save it to the filename...
```
echo UEsDBBQAAAAIb0NhbkRhL0RhdWdodGVyc0Ny
b3du7XpnVBNKu27oXURBihQRpIUmTUKLoICg9BLpIkqV3ntRQCI....CLIP...." | base64 -d | filename
``` 
...and run a `file` comand on it it would return `ZIP-File`

> What is the filetype of the received attachment which helped to continue the investigation? (1 points): `.zip`

If we unpack the attachment we get 3 Files, a JPEG, PDF and XLSX.
![](/assets/images/2022-05-25-07-27-22.png)

... and here is where CTF-y part begins.

# The CTF-y part
## Running exiftool
We can find the author running `exiftool` on the PDF file.
![](/assets/images/2022-05-25-08-51-26.png)

> What is the name of the malicious actor? (2 points): Pestero Negeja

## Analyzing XLSX and Parsing XML
`.xlsx` can be unzipped as any other `.zip` file.
Checking files in unzipped, i've found following text
![](/assets/images/2022-05-25-11-19-33.png)

Decoding the Base64 we can find location that BTLO is asking for
```
sansforensics@siftworkstation: ~/Desktop/cases_CTF/CoCanDa/PuzzleToCoCanDa/xl
$ echo "VGhlIE1hcnRpYW4gQ29sb255LCBCZXNpZGUgSW50ZXJwbGFuZXRhcnkgU3BhY2Vwb3J0Lg==" | base64 -d && echo ""
The Martian Colony, Beside Interplanetary Spaceport.
```
> What is the location of the attacker in this Universe? (2 points): The Martian Colony, Beside Interplanetary Spaceport

# The Last question
It's might be `pashter.com`.
> What could be the probable C&C domain to control the attacker’s autonomous bots? (2 points): pashter.com

# References
https://packagecontrol.io/packages/Email%20Header
https://www.iana.org/assignments/message-headers/message-headers.xhtml
https://en.wikipedia.org/wiki/Bounce_address
https://datatracker.ietf.org/doc/html/rfc4021#page-17
https://gnome.pages.gitlab.gnome.org/libxml2/xmllint.html
