---
title: Server-side request forgery (SSRF)
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-24 09:00:00 +0200
categories: [Web Application, Server-side Request Forgery]
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
This post/writeup is all about the Server-side request forgery (SSRF).

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/ssrf) Labs, but i do intent do throw other labs and writeups here as well.

## TOC

- [Intro](#intro)
  - [TOC](#toc)

# Basic SSRF against the local server
>  This lab has a stock check feature which fetches data from an internal system.
> 
> To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos. 


We can notice that we can check the stock of any product.

![picture 9](/assets/images/f95e835f5f9778a4c60c22a5f1129cd645138598cda8358b1a650d3150737171.png)  

As it can be seen above, it seems like a normal response, however let's check how this looks like in Burp.

![picture 10](/assets/images/b5c24ed135a8ce54123b2731f334fd07daa5052b15be8a8a277a555b9b76a79b.png)  

We can see that we have URL in the POST body and response that corresponds with the one seen on the webpage, but it looks like we can trigger request to any url from the client, so let us do exactly that.

![picture 11](/assets/images/49b9cc3e08804e2281a5908a2efa478a2673d059cfdf956c981e82d2b355efe5.png)  

As seen above, our request does show contents of the `/admin` page and the path to delete the `carlos` user in order to solve the lab is served on a silver plate ==> `/admin/delete?username=carlos`

![picture 12](/assets/images/b0da69d0c1bc7e62d02a3094970a02468c9f8251c37ddec965aede45f6d0bddb.png)  

# Basic SSRF against another back-end system
>  This lab has a stock check feature which fetches data from an internal system.
> 
> To solve the lab, use the stock check functionality to scan the internal 192.168.0.X range for an admin interface on port 8080, then use it to delete the user carlos. 

We have same "Check stock" option, same is in the previous exercise:

![picture 13](/assets/images/40774c57feb38f2008bf422666107671983671c09a64cbe097244eb69f9f4a40.png)  

Vulnerability also appears to be the same, we just now don't know on which IP the `admin` interface resides. This is something where `Burp Intruder` might help us find that out.

![picture 14](/assets/images/02a34ba98c519275ed1e137ceb13fc99552315319ca05824d6a8ade735d97f5d.png)  

Use `Numbers` as a payload set.

![picture 15](/assets/images/1ad0cbb12ca88cc21b60cf645f8aa75536f7d16d2c0a9d08dabdeb825df02a47.png)  

I've got a git at `.189`

![picture 16](/assets/images/30bf7a6e360043f4dee518e8e8c03d4ddb3978636f60b0725341bbdeae3321e4.png)  

Now delete `carlos` in order to solve the lab `/admin/delete?username=carlos`

# SSRF with blacklist-based input filter

# SSRF with whitelist-based input filter

# Blind SSRF with out-of-band detection

# SSRF with filter bypass via open redirection vulnerability

# Blind SSRF with Shellshock exploitation