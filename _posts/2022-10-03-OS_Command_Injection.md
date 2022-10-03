---
title: OS Command Injection
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-03 09:00:00 +0200
categories: [Web Application, OS Command Injection]
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
This post/writeup is all about the OS Command Injection vulnerabilities.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/os-command-injection) Labs, but i do intent do throw other labs and writeups here as well.

# OS command injection, simple case
>  This lab contains an OS command injection vulnerability in the product stock checker.
> 
> The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.
> 
> To solve the lab, execute the whoami command to determine the name of the current user. 

# Blind OS command injection with time delays
>  This lab contains a blind OS command injection vulnerability in the feedback function.
> 
> The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response.
> 
> To solve the lab, exploit the blind OS command injection vulnerability to cause a 10 second delay. 

# Blind OS command injection with output redirection

>  This lab contains a blind OS command injection vulnerability in the feedback function.
> 
> The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response. However, you can use output redirection to capture the output from the command. There is a writable folder at:
`/var/www/images/`

The application serves the images for the product catalog from this location. You can redirect the output from the injected command to a file in this folder, and then use the image loading URL to retrieve the contents of the file.

To solve the lab, execute the whoami command and retrieve the output. 

# Blind OS command injection with out-of-band interaction

>  This lab contains a blind OS command injection vulnerability in the feedback function.
> 
> The application executes a shell command containing the user-supplied details. The command is executed asynchronously and has no effect on the application's response. It is not possible to redirect output into a location that you can access. However, you can trigger out-of-band interactions with an external domain.
> 
> To solve the lab, exploit the blind OS command injection vulnerability to issue a DNS lookup to Burp Collaborator. 

# Blind OS command injection with out-of-band data exfiltration

>  This lab contains a blind OS command injection vulnerability in the feedback function.
> 
> The application executes a shell command containing the user-supplied details. The command is executed asynchronously and has no effect on the application's response. It is not possible to redirect output into a location that you can access. However, you can trigger out-of-band interactions with an external domain.
> 
> To solve the lab, execute the whoami command and exfiltrate the output via a DNS query to Burp Collaborator. You will need to enter the name of the current user to complete the lab. 