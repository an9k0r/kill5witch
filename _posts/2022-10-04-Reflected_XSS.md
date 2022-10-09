---
title: Reflected Cross-Site Scripting (XSS)
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-04 09:00:00 +0200
categories: [Web Application, Cross Site Scripting]
tags: [Notes, Web Application, Portswigger]
math: true
mermaid: true
image:
  src: 
  width: 694
  height: 515
  alt: image alternative text
---
# Intro
This post is dedicated to XSS related Labs at [Portswigger Web Academy](https://portswigger.net/web-security/cross-site-scripting/reflected)

# Finding a XSS

- Test every endpoint
- Submit random alphanumeric values. 
- Determine the reflection context.
  - XSS between HTML tags
  - XSS in HTML tag attributes
  - XSS into JavaScript
  - XSS via client-side template injection
- Test a candidate payload.
- Test alternative payloads.
- Test the attack in a browser. 

Reference: https://portswigger.net/web-security/cross-site-scripting/reflected

# Labs
## Portswigger
| Lab's name |
| --- |
|[Reflected XSS into HTML context with nothing encoded](#reflected-xss-into-html-context-with-nothing-encoded)
|[Reflected XSS into attribute with angle brackets HTML-encoded](#reflected-xss-into-attribute-with-angle-brackets-html-encoded)|
|[Reflected XSS in canonical link tag](#reflected-xss-in-canonical-link-tag)|
|[Reflected XSS into a JavaScript string with single quote and backslash escaped](#reflected-xss-into-a-javascript-string-with-single-quote-and-backslash-escaped)|
|[Reflected XSS into a JavaScript string with angle brackets HTML encoded](#reflected-xss-into-a-javascript-string-with-angle-brackets-html-encoded)|
|[Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped](#reflected-xss-into-a-javascript-string-with-angle-brackets-and-double-quotes-html-encoded-and-single-quotes-escaped)|
|[Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped](#reflected-xss-into-a-template-literal-with-angle-brackets-single-double-quotes-backslash-and-backticks-unicode-escaped)|

# Reflected XSS into HTML context with nothing encoded

>  This lab contains a simple reflected cross-site scripting vulnerability in the search functionality.
> 
> To solve the lab, perform a cross-site scripting attack that calls the alert function. 

So we need to trigger an alert.

This payload does the job:
`<script>alert(1)</script>`

![picture 137](/assets/images/542ba984db26b4cad5ed0dcad8e597f9142ebc1585e280b23fbbe28b3e22f0d7.png)  

We've injected `<script>` tag into `<h1>` as there is no sanitization on the server-side (or even client-site) present:

![picture 138](/assets/images/a07620813f7ae7199b8dcdf14a9ff76b146191abd1c16dcd21f01c5ce5a655f1.png)  

# Reflected XSS into attribute with angle brackets HTML-encoded

> This lab contains a reflected cross-site scripting vulnerability in the search blog functionality where angle brackets are HTML-encoded. To solve this lab, perform a cross-site scripting attack that injects an attribute and calls the alert function. 

We can check how our payload is injected by issuing search. I've used simple `testing` string.

![picture 139](/assets/images/5bd306500bd5f94393a3f69fa53e8d016885f5999414cb2d76940512ecb44fc1.png)  

We can use payload like `" onmouseover="alert(1)` to get out of attribute and create our own.

![picture 140](/assets/images/20400f1d9f0e4d31b4342f3373a8799d039a894032cec434b6e96c1cd7d21abf.png)  

# Reflected XSS in canonical link tag

> This lab reflects user input in a canonical link tag and escapes angle brackets.
> 
> To solve the lab, perform a cross-site scripting attack on the home page that injects an attribute that calls the alert function.
> 
> To assist with your exploit, you can assume that the simulated user will press the following key combinations:
> 
> - ALT+SHIFT+X
> 
> - CTRL+ALT+X
> 
> - Alt+X
> 
> Please note that the intended solution to this lab is only possible in **Chrome**. 

Payload used: `?%27accesskey=%27x%27onclick=%27alert(1)`.

We can see the inject here:
![picture 142](/assets/images/b01e96b4d99d2c3a28775093ab821f471665b10c2c9a980508e9fa270cf3e874.png)  


`accesskey="x"` This sets the X key as an access key for the whole page. When a user presses the access key, the alert function is called. 

![picture 141](/assets/images/792fb5109cf165bb2c0294a710f949ef61e1e6b8cc662b03a7d60aa6a4235fb8.png)  

Here another reference: https://security.stackexchange.com/questions/205975/is-xss-in-canonical-link-possible

And Wikipedia on Link Canonical in general and what's its purpose: https://en.wikipedia.org/wiki/Canonical_link_element

# Reflected XSS into a JavaScript string with single quote and backslash escaped

>  This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality. The reflection occurs inside a JavaScript string with single quotes and backslashes escaped.
> 
> To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the alert function. 

I used random string to see where our possible injection might be:

![picture 143](/assets/images/226911741beb4a658ea4b8862cc67b052a0f9b8139b2de7b6d3ad088f16a35b3.png)  

So it's in the Javascript string as the titel already says - obviously. 

Simply escaping using single quote won't do the job as it get's escaped. We can however close the script and run our own using following payload: `</script><script>alert(1)</script>`

![picture 144](/assets/images/4554e2d3893d98d5f12a0a9c352a4b03ce26e28d7b414a1b30870c8ff95c1909.png)  

This is what happens:

![picture 145](/assets/images/07119d3789d2e7b97f8e3fc6f80694eca946225992f8cc753751ac12a582bc3c.png)  

> [Portswigger](https://portswigger.net/web-security/cross-site-scripting/contexts) says: The reason this works is that the browser first performs HTML parsing to identify the page elements including blocks of script, and only later performs JavaScript parsing to understand and execute the embedded scripts.

Who would have thought, huh!! :)

# Reflected XSS into a JavaScript string with angle brackets HTML encoded

> This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets are encoded. The reflection occurs inside a JavaScript string. To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the alert function. 

Now we're injection again into String in Javascript, however if we try following payload as in the lab before `</script><script>alert(1)</script>`, then we'll see this:

![picture 146](/assets/images/3d1c973e5ba6e8a6c12708fa007ebe98dcb0dd72444f19bc4684d6f3c890a406.png)  

... we didn't get far, did we?

We're still lucky as now single quote does not get encoded.

This is the payload that i've used `'; alert(1) //`, but this would also work `'-alert(1)-'`

![picture 147](/assets/images/e295417b3e8bc22d384570a0284a46e62cbb6cfc99a8b165f5e7bf113e25cea3.png)  

# Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

>  This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets and double are HTML encoded and single quotes are escaped.
> 
> To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the alert function. 

I've just entered the string `qwr`to see where it does get reflected:
![picture 148](/assets/images/3fe5136d8fa95c791327538f37d92301bf4d81fa1f3fc67e0c538a2d66b19739.png)  

Let's try with string `qw/-#\"r`.
We'get `'qw/-#\\&quot;r'` in response. We get escape on single quote and double quote HTML encoded. If we prepend backslash we can escape the escaped backslash for single quote: 
- `\'-alert(1)//` which becomes `var searchTerms = '\\'-alert(1)//';`

![picture 149](/assets/images/7bad8c0b297a166149f672afade03c999f4bfb22396a76abf356c404dfd213a8.png)  

# Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped

> This lab contains a reflected cross-site scripting vulnerability in the search blog functionality. The reflection occurs inside a template string with angle brackets, single, and double quotes HTML encoded, and backticks escaped. To solve this lab, perform a cross-site scripting attack that calls the alert function inside the template string. 

Enter `<>"'()` 

![picture 151](/assets/images/577d5b42048d2a6fdf67a321c178d4554e44e4cb3c74e430c53e5d0106202e0c.png)  

Since we have injection into JavaScript Template literal, we don't need to terminat, but can use `${..}` like `${alert(document.domain)}` which also is solution for this lab.

![picture 152](/assets/images/d9052148dfffc6c7dd119da5787fb33f9b439d8ae53f20590385e2dce7cdabb1.png)  
