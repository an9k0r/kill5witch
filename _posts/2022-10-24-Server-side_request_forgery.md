---
title: (Portswigger/WebAcademy) - Server-side request forgery (SSRF)
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
- [Basic SSRF against the local server](#basic-ssrf-against-the-local-server)
- [Basic SSRF against another back-end system](#basic-ssrf-against-another-back-end-system)
- [SSRF with blacklist-based input filter](#ssrf-with-blacklist-based-input-filter)
- [SSRF with whitelist-based input filter](#ssrf-with-whitelist-based-input-filter)
- [SSRF with filter bypass via open redirection vulnerability](#ssrf-with-filter-bypass-via-open-redirection-vulnerability)
- [Blind SSRF with out-of-band detection](#blind-ssrf-with-out-of-band-detection)
- [Blind SSRF with Shellshock exploitation](#blind-ssrf-with-shellshock-exploitation)

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

I've got a hit at `.189`:

![picture 16](/assets/images/30bf7a6e360043f4dee518e8e8c03d4ddb3978636f60b0725341bbdeae3321e4.png)  

Now delete `carlos` in order to solve the lab `/admin/delete?username=carlos`

# SSRF with blacklist-based input filter
>  This lab has a stock check feature which fetches data from an internal system.
> 
> To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos.
> 
> The developer has deployed two weak anti-SSRF defenses that you will need to bypass. 

Like the previous lab, start the burp in the background and check the stock as in previous labs.

![picture 78](/assets/images/291dbd57ae44152e9b0a8f49f911d59ae8fe579bd20bda0d5a9c481f7ffc8246.png)  

We can enter `http://127.1` and observe that side will get loaded

![picture 79](/assets/images/1f92e0498a5c64d31a71a0809a3af52e8d67d70da2e7d1ec2a1e0133c0d41b1c.png)  

If we add `http://127.1/admin`, we would see a `"External stock check blocked for security reasons"` Error, so admin appears to be on a blacklist.

Double encoding does the job.

![picture 80](/assets/images/05adb953a3787f02fc6907cd0ef6304370bb16cd387ea5188a96ab8106372eba.png)  

In order to solve the lab, following request was issued
```
stockApi=http%3a//127.1/%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65/delete?username=carlos
```

# SSRF with whitelist-based input filter

>  This lab has a stock check feature which fetches data from an internal system.
> 
> To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos.
> 
> The developer has deployed an anti-SSRF defense you will need to bypass. 

In this lab, we again have to exploit the `Check Stock` function.

Let's do that and observe the request in Burp.

![picture 81](/assets/images/3013024963805842cfde52e95bfec20ef93aa6128845a34901dd14783b96acd5.png)  

Send the request to repeater and try some payload like `http://127.0.0.1`:

![picture 82](/assets/images/495347c808d7a01f2d7a7e44de5e7cc2dc6b216edb0e8a7f0d6f166fbe0571a6.png)  

So we need to keep `stock.weliketoshop.net` in the SSRF Request.

Adding a parameter or puting the `stock.weliketoshop.net` after `#` did not work.

- `stockApi=http://e@stock.weliketoshop.net` = WORKS
- `http://127.0.0.1e@stock.weliketoshop.net` still WORKS

- `stockApi=http://127.0.0.1%25%32%33@stock.weliketoshop.net` WORKS where `%25%32%33` is just double-encoded `#`
- `stockApi=http://127.0.0.1%25%32%33@stock.weliketoshop.net/admin` WORKS

![picture 83](/assets/images/07f136382e11b3c62d64e03b1e65b06ab7048281a09c4bdbd8c2eb5eb315e3f3.png)  

Now it should be self-explainatory what to do to delete `carlos`.

# SSRF with filter bypass via open redirection vulnerability

>  This lab has a stock check feature which fetches data from an internal system.
> 
> To solve the lab, change the stock check URL to access the admin interface at http://192.168.0.12:8080/admin and delete the user carlos.
> 
> The stock checker has been restricted to only access the local application, so you will need to find an open redirect affecting the application first. 

Just like before, start the lab, choose a product, click on `Check stock` while having Burp open in the background. Let's examine the request:

![picture 84](/assets/images/e705b06965b454b5aafc534d84e5c3b217d9f5c190d2cf8d0097c4e987d5df69.png)  

While checking the application following comes up
```
/product/nextProduct?currentProductId=1&path=/product?productId=2 
```

![picture 87](/assets/images/6ed54be6f0d15190f76bf5c2eac509d07b3b48277829f939d5375f9d78a840ca.png)  

We've gotten into the `admin` portal! Can we delete `carlos`?

Yes we can!

![picture 86](/assets/images/d177b8b941032f170ac4786026d88382b79398829e61d36e48f65d56aeb56a3d.png)  


# Blind SSRF with out-of-band detection
> Collaborator is needed which is why this lab will be done later at some point

# Blind SSRF with Shellshock exploitation
> Collaborator is needed which is why this lab will be done later at some point