---
title: (Portswigger/WebAcademy) - Insecure Deserialization
 vulnerabilities
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2023-01-15 09:00:00 +0200
categories: [Web Application, Insecure Deserialization]
tags: [Notes, Insecure Deserialization, Portswigger]
math: true
mermaid: true
image:
  src: 
  width: 694
  height: 515
  alt: image alternative text
---
# Intro
This post/writeup is all about the insecure deserialization vulnerabilities.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/deserialization) Labs, but i do intent do throw other labs and writeups here as well.

To learn more on the topic, please visit the article linked above at Portswigger's.

# Modifying serialized objects
> This lab uses a serialization-based session mechanism and is vulnerable to privilege escalation as a result. To solve the lab, edit the serialized object in the session cookie to exploit this vulnerability and gain administrative privileges. Then, delete the user carlos.
> 
> You can log in to your own account using the following credentials: wiener:peter

# Modifying serialized data types
> This lab uses a serialization-based session mechanism and is vulnerable to authentication bypass as a result. To solve the lab, edit the serialized object in the session cookie to access the administrator account. Then, delete the user carlos.
> 
> You can log in to your own account using the following credentials: wiener:peter

# Using application functionality to exploit insecure deserialization
> This lab uses a serialization-based session mechanism. A certain feature invokes a dangerous method on data provided in a serialized object. To solve the lab, edit the serialized object in the session cookie and use it to delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter
> 
> You also have access to a backup account: gregg:rosebud

# Arbitrary object injection in PHP
> This lab uses a serialization-based session mechanism and is vulnerable to arbitrary object injection as a result. To solve the lab, create and inject a malicious serialized object to delete the morale.txt file from Carlos's home directory. You will need to obtain source code access to solve this lab.
> 
> You can log in to your own account using the following credentials: wiener:peter

# Exploiting Java deserialization with Apache Commons
> This lab uses a serialization-based session mechanism and loads the Apache Commons Collections library. Although you don't have source code access, you can still exploit this lab using pre-built gadget chains.
> 
> To solve the lab, use a third-party tool to generate a malicious serialized object containing a remote code execution payload. Then, pass this object into the website to delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter

# Exploiting PHP deserialization with a pre-built gadget chain
> This lab has a serialization-based session mechanism that uses a signed cookie. It also uses a common PHP framework. Although you don't have source code access, you can still exploit this lab's insecure deserialization using pre-built gadget chains.
> 
> To solve the lab, identify the target framework then use a third-party tool to generate a malicious serialized object containing a remote code execution payload. Then, work out how to generate a valid signed cookie containing your malicious object. Finally, pass this into the website to delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter

# Exploiting Ruby deserialization using a documented gadget chain
> This lab uses a serialization-based session mechanism and the Ruby on Rails framework. There are documented exploits that enable remote code execution via a gadget chain in this framework.
> 
> To solve the lab, find a documented exploit and adapt it to create a malicious serialized object containing a remote code execution payload. Then, pass this object into the website to delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter

# Developing a custom gadget chain for Java deserialization
> This lab uses a serialization-based session mechanism. If you can construct a suitable gadget chain, you can exploit this lab's insecure deserialization to obtain the administrator's password.
> 
> To solve the lab, gain access to the source code and use it to construct a gadget chain to obtain the administrator's password. Then, log in as the administrator and delete carlos.
> 
> You can log in to your own account using the following credentials: wiener:peter
> 
> Note that solving this lab requires basic familiarity with another topic that we've covered on the Web Security Academy.

# Developing a custom gadget chain for PHP deserialization
> This lab uses a serialization-based session mechanism. By deploying a custom gadget chain, you can exploit its insecure deserialization to achieve remote code execution. To solve the lab, delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter

# Using PHAR deserialization to deploy a custom gadget chain
> This lab does not explicitly use deserialization. However, if you combine PHAR deserialization with other advanced hacking techniques, you can still achieve remote code execution via a custom gadget chain.
> 
> To solve the lab, delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter



