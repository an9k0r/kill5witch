---
title: Information Disclosure
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-07 09:00:00 +0200
categories: [Web Application, Information Disclosure]
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
This post/writeup is all about the Bussines Logic Vulnerabilities.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/information-disclosure) Labs, but i do intent do throw other labs and writeups here as well.

Information disclosure is all about disclosing information that was not intended to be exposed to user, like debug page for example or leaks data that is intended for privileged or other users.

# Information disclosure in error messages
> This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework. To solve the lab, obtain and submit the version number of this framework. 

This lab is all about the forcing an error. I've done it in parameter

![picture 223](/assets/images/3ce755f5b9c05a9cfff053cf5583d1da9a234fa9e30166581ceb77b11fed9a83.png)  

We have the version and framework listed above

# Information disclosure on debug page
> This lab contains a debug page that discloses sensitive information about the application. To solve the lab, obtain and submit the SECRET_KEY environment variable. 

Always keep an eye on the `Site map`. We can see that site leaks it's `phpinfo` page.

![picture 224](/assets/images/79977051feb1ce8b0d87d4aeba543ab48290d62859b6c78bc6546ac26d6a646e.png) 

`SECRET_KEY` can be found inside:

![picture 225](/assets/images/39f4573489aed7c0ab3e9d6faa6afb93de4296834be26afc49d293cbd6e6d968.png)  

# Source code disclosure via backup files
> This lab leaks its source code via backup files in a hidden directory. To solve the lab, identify and submit the database password, which is hard-coded in the leaked source code. 

One of the low-hanging fruits is `robots.txt`. We might always find pages there that are not supposed to be listed by search engines like google, bing and others of their kind.

![picture 226](/assets/images/0077f7d7389bf5a61c36bb14ff217a7a35e6a7071bcc4d65c1959ca8ba6cba65.png)  

We can even see a directory listing on `/backup`:

![picture 227](/assets/images/b9ba76bd3414aa724c7364d4460392aa3224c1c238905cc3188687e332493315.png)  

We can find information about the database in the `ProductTemplate.java.bak` which has been found before.

![picture 228](/assets/images/da0d81e8c9f03f6579cfbb458a34bc32f64321661866e3930f7e578d79f816dc.png)  


# Authentication bypass via information disclosure
> This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.
> 
> To solve the lab, obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete Carlos's account.
> 
> You can log in to your own account using the following credentials: wiener:peter 

If we login using `wiener:peter` and go to `/admin` page we'd see following message:

![picture 229](/assets/images/d93509a9922750e1e8221a8f5511fe987962912088fe2520bb23ed93987c1054.png)  

If we send HTTP TRACE to `/admin` another header name reveals `X-Custom-IP-Authorization: 84.138.106.72`

![picture 230](/assets/images/35bca5f0170a16f5880ce0f45b22dfc9c35a74c3b602d447df49e0cb257aed2d.png)  

If we add the header and point to localhost's IP `127.0.0.1` we get onto the `/admin` panel.

![picture 231](/assets/images/407e7af790b3db3129889b3e2b2e3ec5dac6cded433eee133dcc4d6628d2c650.png)  

We can either delete carlos from here with another GET request to `/admin/delete?username=carlos` or use `Match and replace` and get it done over browser.

![picture 232](/assets/images/0d1eef04894fadbd00c5d08d9af98afb7cf823fd2cf8a73a5f6b8b882af89839.png)  

# Information disclosure in version control history
> This lab discloses sensitive information via its version control history. To solve the lab, obtain the password for the administrator user then log in and delete Carlos's account. 

It's all about the GIT!! ;)

As usual we start with the shop. To solve the lab we hovewer need to check what we can found in the background.

![picture 233](/assets/images/a1a512c2109153f432a9af38173b20015e1974d0dba20d2c1cdf779bb5409521.png)

Let's check for `.git` 

![picture 234](/assets/images/6ddc66d359dccc270e0bba8211300b5773bb92ec811662c3c18ddd96d549cda3.png)  

I've used other GIT tools in the past, but we can simply download the directory recursively using `wget -r https://0a74000a03a77c9ec065ef4100520014.web-security-academy.net/.git`.

Easiest way to check the commit information is by `git show` in the `.git` directory.

![picture 235](/assets/images/1c98afdfe73200e139aaf2b4e27b3485acb2bdbba9df6f001cb2aab3ac828a5e.png)  

As the password change is the only change it is very easy to spot.

We can use the found password with `administrator` user to login and delete `carlos` to solve the lab.

![picture 236](/assets/images/817bcb22550adfc747d4ce378716e2e8f9e3de407649c75d43978244cebbc20a.png)