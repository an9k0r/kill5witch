---
title: (Portswigger/WebAcademy) - Web Cache Poisoning (Unkeyed Inputs)
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-27 09:00:00 +0200
categories: [Web Application, Web Cache Poisoning]
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
This post/writeup is all about the Web Cache Poisoning.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/web-cache-poisoning) Labs, but i do intent do throw other labs and writeups here as well.

To learn more on the topic, please visit the article linked above at Portswigger's. I also recommend reading [Practical Web Cache Poisoning](https://portswigger.net/research/practical-web-cache-poisoning) Article.

## TOC
- [Intro](#intro)
  - [TOC](#toc)
- [Web cache poisoning with an unkeyed header](#web-cache-poisoning-with-an-unkeyed-header)
- [Web cache poisoning with an unkeyed cookie](#web-cache-poisoning-with-an-unkeyed-cookie)
- [Web cache poisoning with multiple headers](#web-cache-poisoning-with-multiple-headers)
- [Targeted web cache poisoning using an unknown header](#targeted-web-cache-poisoning-using-an-unknown-header)
- [Web cache poisoning to exploit a DOM vulnerability via a cache with strict cacheability criteria](#web-cache-poisoning-to-exploit-a-dom-vulnerability-via-a-cache-with-strict-cacheability-criteria)
- [Combining web cache poisoning vulnerabilities](#combining-web-cache-poisoning-vulnerabilities)


# Web cache poisoning with an unkeyed header
> This lab is vulnerable to web cache poisoning because it handles input from an unkeyed header in an unsafe way. An unsuspecting user regularly visits the site's home page. To solve this lab, poison the cache with a response that executes **alert(document.cookie)** in the visitor's browser.

We start with the Shop page

![picture 1](/assets/images/99b6d74cbc49f2f157ef80671d80a1eb2fa93999d05c1a929c8e9857c29a2013.png)  

Let's check the request(s) in Burp.

We can already tell that we're caching is active on the webserver (`X-Cache` header is present along with `Age` and `Cache-Control'`)

![picture 2](/assets/images/d4f64a140cf773bb3acd26fc14e7c931cfeb7e9bbbfab3380d351bb931b6f4d6.png)  

Can we poison the Cache?

![picture 5](/assets/images/9442b0b342755a0a0a7df3e0515175fdbe9cc01ce35495ef331b23a1f99a2846.png)  

Yes we can, using `X-Forwarded-Host`. Make sure to get rid of path that get's appended afterwards (mind the `#`).

![picture 6](/assets/images/0deb18bc0fa9db743a57685dc089e81403ded4261655f5e804d82485dce1fd84.png)  

Lab has been solved. 
PS: You can also check the Access Logs to see if `Victim` has been connecting to exploit server or not. 

# Web cache poisoning with an unkeyed cookie
> This lab is vulnerable to web cache poisoning because cookies aren't included in the cache key. An unsuspecting user regularly visits the site's home page. To solve this lab, poison the cache with a response that executes **alert(1)** in the visitor's browser.

The page looks same as in the lab before. If we take a look at the requests we'll see `fehost` cookie, and it's value gets reflected to the response.

![picture 7](/assets/images/83d4a7d9c7752270956e1207104d455baa5fd5861e51231439e060c442c4e73a.png)  

If we carefully check how the values gets translated into response, we can come up with something like that:

![picture 8](/assets/images/7a3bb8c29e2f09e6dd039f5715e3b7e38e9874f897a89f84574d020eb61790bf.png)  

We just need to trigger `alert(1)` that has been put into cache. TTL is again 30 seconds.

![picture 9](/assets/images/55477d90f589fb4ca5f6a079604311aa45466fae727cdd2f33f1526e7de45c35.png)  

# Web cache poisoning with multiple headers
> This lab contains a web cache poisoning vulnerability that is only exploitable when you use multiple headers to craft a malicious request. A user visits the home page roughly once a minute. To solve this lab, poison the cache with a response that executes alert(document.cookie) in the visitor's browser.

Page looks exacly the same as in the previous labs. This lab however needed more fiddling with the headrers and was not so straightforward as the previous labs. 

Main difference here is when `X-Forwarded-Scheme: http` is used, server answers with 302

![picture 10](/assets/images/43f1158967cea560a0205df71d8f295b2ad7dc1afd9ba2edd393c77175127060.png)  

If we use `X-Forwarded-Host` as well, we get `302` redirection to our exploit

![picture 12](/assets/images/7a5c8c97ff8e85b3a47baa2d0c831456dfd0acd4ba5845abe6da729ce08fb8b5.png)  

Craft the payload accordingly (`alert(document.cookie)`), and also make sure to disable the path that get's appended to `Location`.

![picture 11](/assets/images/74617ca1eb2e2993b2c6212d803f9268d6b70f4040b13459e7f02d48e3ef2dea.png)  

You can use Access Logs to see if `Victim` has made a connection or not. If everything was done right, lab should be solved!

# Targeted web cache poisoning using an unknown header
> This lab is vulnerable to web cache poisoning. A victim user will view any comments that you post. To solve this lab, you need to poison the cache with a response that executes alert(document.cookie) in the visitor's browser. However, you also need to make sure that the response is served to the specific subset of users to which the intended victim belongs.

In this lab when we load the page, we see `Vary` header in the response, which implies that caching should be `User Agent` based. At least that was my understanding when solving this particular lab.

![picture 13](/assets/images/0fd114d5540a97bac87704152244b9449f078595daa2591b84e039774422c620.png)  

If we start `Param-miner` Burp extension, it should return 2 headers back to us. Origin is most like not unkeyed though in the cache.

```
Loaded Param Miner v1.4d
Updating active thread pool size to 8
Queued 1 attacks
Initiating header bruteforce on 0ab600840368b45fc045a7e8004f00ee.web-security-academy.net
Identified parameter on 0ab600840368b45fc045a7e8004f00ee.web-security-academy.net: x-host
Identified parameter on 0ab600840368b45fc045a7e8004f00ee.web-security-academy.net: origin~https://%s.%h
Identified parameter on 0ab600840368b45fc045a7e8004f00ee.web-security-academy.net: origin
```

And indeed, `X-Host` does reflect itself in the response.

![picture 1](/assets/images/8cbad2c72b174fe4a5a349188531aa89d388e11e85a3fac06098c4fa02dc77d0.png)  

Now we'll need a XSS to effiecently exploit our victim as we need to know what User-Agent it is being used.

We can post a comment under any post like a simple image
```
<img src="exploit-server" />
```

You should receive a callback from victim with it's `User-Agent`.

![picture 3](/assets/images/41239b66929315b2770795c46fea11fd050d7dc3d9e3f55aa05bb5693f908547.png)  

Now let's try to poison webcache.

We need to change payload accordingly on our exploit server => `alert(document.cookie)`. When we poison the Cache, don't forget to change the `User-Agent`.

![picture 4](/assets/images/75faa5e92a0472aafb5e5874db61624af32294b85788be5c06b8fa151a4e40f1.png)  

# Web cache poisoning to exploit a DOM vulnerability via a cache with strict cacheability criteria
> This lab contains a DOM-based vulnerability that can be exploited as part of a web cache poisoning attack. A user visits the home page roughly once a minute. Note that the cache used by this lab has stricter criteria for deciding which responses are cacheable, so you will need to study the cache behavior closely.
> 
> To solve the lab, poison the cache with a response that executes alert(document.cookie) in the visitor's browser.

Let's browse the site first. The `Param Miner` extension returned `X-Forwarded-Host` as being reflected in response.

![picture 6](/assets/images/5d28c0244f75a2b1f529c108dcfb6a3ad9e7fcc98a5d4c99301438c004db0484.png)  

It relects itself as value in JSON format, enclosed in `<script>`.

![picture 5](/assets/images/40ca181ed884d1a2401c2f1a81f19a5279abf302f5c037ab41c32a89e8f3aa27.png)  

We need to scroll further down in the source code to find out where the `data.host` is actually being called.

```js
                   <script>
                        initGeoLocate('//' + data.host + '/resources/json/geolocate.json');
                    </script>
```

`data.host` is not being sanitized in any way.

Access log shows that we're hitting the exploit server.
```
84.138.97.163   2022-11-26 12:09:08 +0000 "GET /exploit/?=/resources/json/geolocate.json HTTP/1.1" 200 "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:107.0) Gecko/20100101 Firefox/107.0"
```

If we check the `initGeoLocate` function we will also see that our input is being put directly into `div` as value

![picture 1](/assets/images/73f55c07b90cbbf52bf2638bc310d4462480695ad87a9f3940c65b71e6d00c12.png)  

We also need to bypass CORS, as if we try to load remote resource we'll see CORS error `Cross-Origin Request Blocked`.

```
Access-Control-Allow-Origin: *
```

This is what i've used on the exploit server.

![picture 2](/assets/images/14b8ba5cb50dd51ae071a84a20ed586933b3be45ecfd1acf8e6f30b5319a19eb.png)  

If everything has been done right, we should get an alert in OUR browser. I've poisonied more that 1 product though and the `/` in order to solve the lab!

![picture 3](/assets/images/08cc27547a95f89b2b3244cadcbc2988a91af9d3f749ac9d65fba5e0e086388d.png)  

# Combining web cache poisoning vulnerabilities
> This lab is susceptible to web cache poisoning, but only if you construct a complex exploit chain.
> 
> A user visits the home page roughly once a minute and their language is set to English. To solve this lab, poison the cache with a response that executes `alert(document.cookie)` in the visitor's browser.

This labs needs combining two vulnerabilities togheter where we actually have to poison the cache on two places.

I've started `Param Miner` on `/` and `/?localized=1` (actually on other endpoits well, but the ones mentioned are relevant!)

- `/` returned `X-Forwarded-Host` and `X-original-url`
- `/localised=1` returned `X-original-url`

In order to even find the endpoints/paths i simply browsed the page.

We now know where the potential Web Cache Vulnerability might be. 

![picture 4](/assets/images/44e9214a96c696e8574ca5347aae4e5995de5c1f303531fbf1e951fd8727feaf.png)  

We can notice that host is being reflected being sent into following script

```js
<script>
  initTranslations('//' + data.host + '/resources/json/translations.json');
</script>
```

File `translations.json` looks like this (it was already modified by me, only the `español`)

```
{
    "en": {
        "name": "English"
    },
    "es": {
        "name": "español",
        "translations": {
            "Return to list": "its just on the product page :(",
            "View details": "</a><img src=1 onerror='alert(document.cookie)' />",
            "Description:": "Descripción:"
        }
    },
    "cn": {
        "name": "中文",
        "translations": {
            "Return to list": "返回清單",
            "View details": "查看詳情",
            "Description:": "描述:"
        }
    },
    "ar": {
        "name": "عربى",
        "translations": {
            "Return to list": "العودة إلى القائمة",
            "View details": "عرض التفاصيل",
            "Description:": "وصف:"
        }
    },
    "en-gb": {
        "name": "Proper English",
        "translations": {
            "Return to list": "From whence you came",
            "View details": "Do me the honour of elaborating",
            "Description:": "Pontifications on the subject matter:"
        }
    },
    "ml": {
        "name": "മലയാളം",
        "translations": {
            "Return to list": "ലിസ്റ്റിലേക്ക് മടങ്ങുക",
            "View details": "വിശദാംശങ്ങൾ കാണുക",
            "Description:": "വിവരണം:"
        }
    },
    "hb": {
        "name": "עברית",
        "translations": {
            "Return to list": "חזור לרשימה",
            "View details": "הצג פרטים",
            "Description:": "תיאור:"
        }
    },
    "zl": {
        "name": "Ẕ̻͕̿̊ͤ̍ͅa͙l̗ͧg̮̤̰̘͇ȍ͇͕̳̙͙͉́̅̋̌̅",
        "translations": {
            "Return to list": "Re̹̰̘͉̹̪ͅt̬̫̜ȕͩ͒ͥͥr̃̉͒n ̎͂t͎͖̽͋o͖̟͚͙̲͐ͤͫ̎̓ ̼̟͈̭͉͎̂ͯ̔ͤͤ̏͐ͅliͤ͑ͧ̆̐̈̀sṭ̠̮̰͍̙͒̔͆̈ͤ̅",
            "View details": "V̖̮͙ͅi͇e͙̦w̭̣̫͇̦̬̰ ̓͑̓ͯ̔d͍͂e͚̮͖͍͖̠͙ͮͭ̉ͦ̏͌̆t̙͎̺͉a̳̖͔̱͉̱͑̆̌̃͊ͬi̯͚͙̼̹̮l̖͎͛̈́͒ͅs̒̒ͤ̽̒̀",
            "Description:": "D̳͔e̝ͩ̐ͅsc̗̱̼̤̬̎̓ͪͣͭ̐ͅr̪̝͖̙̱̄̓͌̓̚ip̭̦̭̰̻ͣ̓̽ͨ̚ț̤̝̻i̹̱̟̞͕̓̓ͬ̓ͬ̆ͅon̠͚͕̈́̋̓:"
        }
    },
    "fn": {
        "name": "Suomalainen",
        "translations": {
            "Return to list": "Palaa luetteloon",
            "View details": "Näytä kuvaus",
            "Description:": "Kuvaus:"
        }
    },
    "hw": {
        "name": "Ōlelo Hawaiʻi",
        "translations": {
            "Return to list": "Hoʻi i ka papa inoa",
            "View details": "E nānā i nā kikoʻī",
            "Description:": "ʻO keʻano:"
        }
    },
    "mm": {
        "name": "ဗမာ",
        "translations": {
            "Return to list": "စာရင်းသို့ပြန်သွားသည်",
            "View details": "အသေးစိတ်ကြည့်ရန်",
            "Description:": "ဖော်ပြချက်:"
        }
    }
}
```

Do not forget to add `Access-Control-Allow-Origin: *` into the header.

With the payload above, if we poison `/`, we'll trigger the payload but ONLY if we chose ES language! 

![picture 5](/assets/images/44e9214a96c696e8574ca5347aae4e5995de5c1f303531fbf1e951fd8727feaf.png)  

Now we should be able to trigger an Alert. 

To trigger an alert at `victim`'s, We'd need to poison cache on another spot. If we set `x-original-url: /setlang\es`

![picture 6](/assets/images/ba77ed0921bceaa9c0e2e3f435d9a50b4072767a0b0768f7a05a5d436fa60784.png)  

The `/setlang\es` part was the one that needed some fiddling around.

![picture 7](/assets/images/2ac2e7c0b26bdb3dd944309cc80aee842a8fbe46354073a2f8430fb6c8349d34.png)  

![picture 8](/assets/images/510a0d3b6fccba4bff844a6bf70fd060b2957a0811e3e79c88068db41f58ee6f.png)  

If we twist the slash, webserver will autocorrect. We can confirm that `302` requests are cache-ble and autocorrection is doing us a favour.

If we've done everything right and poisoned the cache which will lure all users to spanish page, we should have solced the lab.

![picture 9](/assets/images/62e28004d8ae8dc988739520c8b0a47420b5ea8be9bdb921d9103cadb3fdcef7.png)  
