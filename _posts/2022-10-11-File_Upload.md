---
title: File Upload Vulnerabilities
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-09 09:00:00 +0200
categories: [Web Application, File Upload Vulnerabilities]
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
This post/writeup is all about the File Upload Vulnerabilities.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/file-upload) Labs, but i do intent do throw other labs and writeups here as well.

## TOC
- [Intro](#intro)
  - [TOC](#toc)
- [Remote code execution via web shell upload](#remote-code-execution-via-web-shell-upload)
- [Web shell upload via Content-Type restriction bypass](#web-shell-upload-via-content-type-restriction-bypass)
- [Web shell upload via path traversal](#web-shell-upload-via-path-traversal)
- [Web shell upload via extension blacklist bypass](#web-shell-upload-via-extension-blacklist-bypass)
- [Web shell upload via obfuscated file extension](#web-shell-upload-via-obfuscated-file-extension)
- [Remote code execution via polyglot web shell upload](#remote-code-execution-via-polyglot-web-shell-upload)
- [Web shell upload via race condition](#web-shell-upload-via-race-condition)

# Remote code execution via web shell upload
>  This lab contains a vulnerable image upload function. It doesn't perform any validation on the files users upload before storing them on the server's filesystem.
> 
> To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.
> 
> You can log in to your own account using the following credentials: `wiener:peter`

For this lab we should use simple PHP script that only prints the `/home/carlos/secret`. I will use same script that`s been provided in [File Upload](https://portswigger.net/web-security/file-upload) article.

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

So let us upload the file

![picture 1](/assets/images/be2a5a585ea3c523a9a27691a511fd494dad5b245a88125ac2433492d1714c78.png)  

Our malicious `PHP` file has been uploaded

![picture 2](/assets/images/4b2ab4b29b3dcea1e92fd58153a8372064312a6c976225c09540b546edf2cc13.png)  

As image gets loaded to frontend, we can simply check the source code for location (e.g., through `Inspect`)

![picture 3](/assets/images/063450a38f3b77acf4fe26f69766d8dc629508de891d6a8bae3c4fa86e418fc4.png)  

We can read the `secret`. We should submit it to solve the lab.

![picture 4](/assets/images/ec6e63076681d862f31787ef3849353444bb7cf491e77a7e89e8200d536f80ed.png)  

# Web shell upload via Content-Type restriction bypass
>  This lab contains a vulnerable image upload function. It attempts to prevent users from uploading unexpected file types, but relies on checking user-controllable input to verify this.
> 
> To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.
> 
> You can log in to your own account using the following credentials: `wiener:peter` 

Let us login using provided credentials `wiener:peter`.

Then try to upload malicious PHP file:
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

Don't forget to have BURP opened so we can tamper with the request later...

![picture 5](/assets/images/a75900836d150604947bb69ed6639ba626a5dac6e0dcb2405b2e6f460cd48fe7.png)  

We get following message: `Sorry, file type text/php is not allowed Only image/jpeg and image/png are allowed Sorry, there was an error uploading your file.`

Let's check the request in Burp, send it to repeater and adjust the `Content-Type` to `image/jpeg`

![picture 6](/assets/images/82ba8b90032abd288f9e1aeb30ba5306236965bd2db0ad554e8a10b6948cbf30.png)  

As it can be observed in the response above, file was succesfuly uploaded.

Our malicious PHP's location is again leaked on the front-end and it's in the same location as in previous lab.

![picture 7](/assets/images/6d92a010bb2ec532f5559d468e07038573e24e3cb5f4d255c39fd6fd54d1fabe.png)  

Submit the flag to solve the lab.

# Web shell upload via path traversal
>  This lab contains a vulnerable image upload function. The server is configured to prevent execution of user-supplied files, but this restriction can be bypassed by exploiting a secondary vulnerability.
> 
> To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.
> 
> You can log in to your own account using the following credentials: `wiener:peter`

For this lab we only need to find an executable directory through `Directory Traversal`. 

Let's login and upload the malicious PHP file (same as previous lab) while having Burp opened in the background.

![picture 8](/assets/images/f1ba7d4f01a39136a4491fefda849ad737beb1d6f4cce4dde5bc01dbb1edf8e3.png)  

File was uploaded, now let's try to execute the file (same location as previous lab).

Contents will be printed but not executed.

![picture 9](/assets/images/40a11b65d54fced926b74ab89d1cb4044375a45fc39c6cbaaef7565221e7d7d4.png)  

`Directory Traversal` took few tries. I've put both relevant requests to repeater - the upload one and the file read one. What has worked was encoding `/` with `%2f`

![picture 10](/assets/images/9bc393447cc7803f1b2314b2012868c9a4cef2f97746fab6584745e94be6cdc8.png)  

We can read the file and lab's secret flag in `/files` now and not in `/files/avatars` anymore as we've traversed one directory backwards.

![picture 11](/assets/images/c2c5d9a212076140f6cd81b634d4598c7b4f791ab786be62036b0f6951585b6b.png)  

Submit the flag to solve the lab.

# Web shell upload via extension blacklist bypass

> This lab contains a vulnerable image upload function. Certain file extensions are blacklisted, but this defense can be bypassed due to a fundamental flaw in the configuration of this blacklist.
> 
> To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.
> 
> You can log in to your own account using the following credentials: `wiener:peter`

Let us login using provided credentials `wiener:peter` and upload some image. We will want to check this in Burp after upload.

![picture 1](/assets/images/20b3ad66e596d08e5209524a486d929d3e40b4362bdbbf66644d53a496faab29.png)  

Find that request and send to repeater. Now uploading from files with `.php5`, `phtml` will work, but PHP code won't execute!

![picture 2](/assets/images/035180133991a221cca3bad9038f735e2553476c71f6c066c170af304d06a790.png)  

We will upload `.htaccess` file which will give directives to apache how to execute certain file types. 

![picture 3](/assets/images/d18c29a94d78b08264b5a25e77de0ccf25ad1c5eeda3be3d0d1c055290bf0678.png)  

If we try to visit the `.htaccess` we would get `403 Forbidden`. 

Let's upload `shell.l33t`

![picture 4](/assets/images/7d9ec990dcd8b8f294adf485eb8ef9ffc5650436d4d82f815abfbe835f12a953.png)  

We should have code execution now.

![picture 5](/assets/images/f7255951798a0507c8d31efd3a751f097044ebba015c25f1f51e3242a53fd316.png)  

We can find and read the flag
![picture 6](/assets/images/628d04ab931099ab8328dc49eaf8c15ae66347b99a1a1e1a2f6e7a1739f8c408.png)  

Alternatively we can use following payload which will display the secret flag:

```php
<?php echo file_get_contents("/home/carlos/secret"); ?>
```

There is great explanation available on youtube https://www.youtube.com/watch?v=b6R_DRT5CqQ 

Keep in mind that `.htaccess` are applied on per-directory basis!

# Web shell upload via obfuscated file extension

> This lab contains a vulnerable image upload function. Certain file extensions are blacklisted, but this defense can be bypassed using a classic obfuscation technique.
> 
> To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.
> 
> You can log in to your own account using the following credentials: `wiener:peter`

Log in using provided credentials `wiener:peter` and upload an image. We will check the request later in Burp.

![picture 7](/assets/images/d3b86e54c95c0cfe0b641f4db871321d47f449d1026880f40f29f8f797b3d36c.png)  

We can break the extension using null byte `%00`

File will upload and we can read the flag.

# Remote code execution via polyglot web shell upload

>   This lab contains a vulnerable image upload function. Although it checks the contents of the file to verify that it is a genuine image, it is still possible to upload and execute server-side code.
> 
> To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner. 
> 
> You can log in to your own account using the following credentials: `wiener:peter`

Start the lab and log in using provided credentials `wiener:peter`

Upload one valid image and check the request in Burp. We can change the filename to something like `shell.php`. Payload used at the end of the image:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

![picture 8](/assets/images/70ff1bcebd2460d698b14f9661b900c8eff8e28ff8ed333ff78c61f0ed213335.png)  

We can submit the flag and solve the lab.

# Web shell upload via race condition
> Lab will be solved at the later time.