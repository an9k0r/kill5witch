---
title: Directory Traversal
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-02 09:00:00 +0200
categories: [Web Application, Directory Traversal]
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
This post/writeup is all about the Directory Traversal vulnerabilities.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/file-path-traversal) Labs, but i do intent do throw other labs and writeups here as well.

# TOC

- [Intro](#intro)
- [TOC](#toc)
- [File path traversal, simple case](#file-path-traversal-simple-case)
- [File path traversal, traversal sequences blocked with absolute path bypass](#file-path-traversal-traversal-sequences-blocked-with-absolute-path-bypass)
- [File path traversal, traversal sequences stripped non-recursively](#file-path-traversal-traversal-sequences-stripped-non-recursively)
- [File path traversal, traversal sequences stripped with superfluous URL-decode](#file-path-traversal-traversal-sequences-stripped-with-superfluous-url-decode)
- [File path traversal, validation of start of path](#file-path-traversal-validation-of-start-of-path)
- [File path traversal, validation of file extension with null byte bypass](#file-path-traversal-validation-of-file-extension-with-null-byte-bypass)


# File path traversal, simple case
>  This lab contains a file path traversal vulnerability in the display of product images.
> 
> To solve the lab, retrieve the contents of the /etc/passwd file. 

If we check the source, we see that images will be loaded through parameter. 

![picture 120](/assets/images/c597993ff6131a61361fa51d4c5bc275921f2a63919e9c24b9418c206759eec9.png)  

If not sanitized corectly, we'll have a directory traversal so let's have a try.

![picture 121](/assets/images/fd668e2eee1f824ea9b6a443dd6e06fb185286db374724535a2a48e2b2337cbe.png)  

Lab has been solved:

![picture 122](/assets/images/846935903fdd2bfb47792706829326402fe31cccaca05d92c573af36fee43c9e.png)  

# File path traversal, traversal sequences blocked with absolute path bypass

>  This lab contains a file path traversal vulnerability in the display of product images.
> 
> The application blocks traversal sequences but treats the supplied filename as being relative to a default working directory.
> 
> To solve the lab, retrieve the contents of the /etc/passwd file. 

Solution - Absolute Path: `/image?filename=/etc/passwd`

![picture 123](/assets/images/284ad5654d2b11062c08b8a45057e13117b96999e46b227114fac5d22138cb2b.png)  

# File path traversal, traversal sequences stripped non-recursively
>  This lab contains a file path traversal vulnerability in the display of product images.
> 
> The application strips path traversal sequences from the user-supplied filename before using it.
> 
> To solve the lab, retrieve the contents of the /etc/passwd file. 

Solution: `/image?filename=....//....//...//....//etc/passwd`

![picture 124](/assets/images/e2b426de55767b8c9ae361bbb734510ada8a9b733bbaac1a4e18a59398779476.png)  

# File path traversal, traversal sequences stripped with superfluous URL-decode
>  This lab contains a file path traversal vulnerability in the display of product images.
> 
> The application blocks input containing path traversal sequences. It then performs a URL-decode of the input before using it.
> 
> To solve the lab, retrieve the contents of the /etc/passwd file. 

Solution: `/image?filename=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc/passwd`

![picture 125](/assets/images/7a8ee885d46ada4e07c55672470647193f56112a0f0c72b3cedd25c32dad8bc1.png)  

# File path traversal, validation of start of path
>  This lab contains a file path traversal vulnerability in the display of product images.
> 
> The application transmits the full file path via a request parameter, and validates that the supplied path starts with the expected folder.
> 
> To solve the lab, retrieve the contents of the /etc/passwd file. 

Solution: `/image?filename=/var/www/images/../../../etc/passwd`

![picture 126](/assets/images/67aacacdae05d85c63bbca9bc9a37f9b303b44e45b66ba7812b5617a83b2204a.png)  

PS: I'm not sure how common this vulnerability is!

# File path traversal, validation of file extension with null byte bypass
>  This lab contains a file path traversal vulnerability in the display of product images.
> 
> The application validates that the supplied filename ends with the expected file extension.
> 
> To solve the lab, retrieve the contents of the /etc/passwd file. 

Solution: `/image?filename=../../../etc/passwd%00.png`

![picture 127](/assets/images/4cbb65998eec18a495dc02c6f1fbc0976e89d7e2acdc140fe6539d2d47119925.png)  
