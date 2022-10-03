---
title: Multi-Factor Authentication (MFA)
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-09-30 09:00:00 +0200
categories: [Web Application, Broken Authentication]
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
This post/writeup is all about the Authentication vulnerabilities or [Broken Authentication](https://owasp.org/www-project-top-ten/2017/A2_2017-Broken_Authentication) if we follow [OWASP](https://owasp.org/) naming scheme. 

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/authentication) Labs, but i do intent do throw other labs and writeups here as well.

> 2FA (two-factor authentication) is based on something you know and something you have and should be implemented in a way so they check the same factor in two/more diferent ways. 

# 2FA simple bypass
>  This lab's two-factor authentication can be bypassed. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, access Carlos's account page.
> 
> Your credentials: wiener:peter
> 
> Victim's credentials carlos:montoya

## Bypassing the 2FA through bad auth. implementation
If we login as `wiener:peter` we recieve an Email.

![picture 69](/assets/images/83e8cc75fdd38530bc6220d81ca82d4502e57263704dbd5b815cea0e503a82c5.png)

If we enter the 4-digit code, we'd get to `my-account` page. 

Problem with this applications authentication implementation is that when we've entered the `username:password` we're already logged in thus skipping the 2FA is entirely possible.

If we enter `carlos:montoya` we'd get asked for 4-digit code BUT if we then just go to `/my-account` we would have bypassed that step. 

If we do just that as described, we'd solve the lab!

![picture 70](/assets/images/090cef16d4e8e0289ea12e63fbfacf7691decea4901aadff14fea1d4110da625.png)

# 2FA broken logic
>  This lab's two-factor authentication is vulnerable due to its flawed logic. To solve the lab, access Carlos's account page.
> 
> Your credentials: wiener:peter
> 
> Victim's username: carlos
> 
> You also have access to the email server to receive your 2FA verification code. 

## Vulnerable 2FA broken logic Enumeration
Let's first login using known working credentials `wiener:peter`. 

After entering credentials we have to enter 4-digit code which we can retrieve from `Email client` with a message like `Hello! Your security code is 1731.`.

Let's inspect both requests now in Burp.

### 1st Request to /login

![picture 73](/assets/images/36ffc8526e65323b4d96cfc5a0cbab71d7c6959c04e1ef0a459dc9442c92217d.png)  

### 2nd Request to /login2

![picture 71](/assets/images/71a95be8fd95228572c5dcfd0649a92eb19fb3d7ab963d94102d048e05ca21b1.png)

There is an unusual cookie `verify=wiener`. 

If we send GET request to `login2` to Burp Repeater. Now few Questions arise:
- Can we simply ask for 2FA token? We should recieve a new code if we use repeater
- Can we also do it for `carlos`? We shouldn't recieve any new code when we use repeater => GET Request to `/login2`
- Can we bruteforce the code? Will we be redirected to `carlos`'s `my-account` or back to login or will we see any errors?

## Vulnerable 2FA broken logic Exploitation

Let's ask for token for `carlos`.

![picture 74](/assets/images/2389d3c2ee13504ae4ae1f0b2088485461c5638df7b34d1252d2bf9b9804b26c.png)  

Send to `Turbo Intruder`:

![picture 75](/assets/images/b98c25a4111bf2de1e2a9b7a1641c89d53fa51f203d56a42e6a797c6b13a4ebe.png)  

We know that we need 302 as result, considering how application reacts on correct MFA-code.

![picture 77](/assets/images/71ba946c41827d30d6a46dd96fe963e87bdd6eec0a93570c513ce59ac792d5b6.png)

Code used in Turbo Intruder. Only digits to 3000 will be used, but it could've been set to 9999
```py
# Find more example scripts at https://github.com/PortSwigger/turbo-intruder/blob/master/resources/examples/default.py
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=3,
                           requestsPerConnection=4,
                           pipeline=False
                           )
    for i in range(3000):
        engine.queue(target.req, '{:d}'.format(i).zfill(4))
        

@MatchStatus(302)
def handleResponse(req, interesting):
    if interesting:
        table.add(req)

```

There was a hit in ca 45 seconds.

![picture 78](/assets/images/3c4d9a2fbc206864cf866d282545fc0862c4a4a9897ee1d180d9daf00ee2326f.png)  

Code was 0810.

If i open same request in the Browser, we're logged in as Carlos

![picture 79](/assets/images/0e57204b9fef5eb4a3d56138c3f16b1f408a6d90cddae0fd25a5f7deffd5cc7b.png)  

# 2FA bypass using a brute-force attack
>  This lab's two-factor authentication is vulnerable to brute-forcing. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, brute-force the 2FA code and access Carlos's account page.
> 
> Victim's credentials: carlos:montoya 

## 2FA Enumeration
Since 2FA mechanismus is pretty much the same as in the previous 2 exercises i'll just describe what happens:
1. POST Request to `login` with username and password
2. We land on `login2` where we have to enter 4-digit token.

![picture 80](/assets/images/618dae7efe87bd4dac2a4ac813f296f8531180a0ee88b5b5c5a151544a932ebc.png)  

Now if there is no Brute-force defense mechanismus on 4-digit PIN we will be able to brute-force it, however we need to bypass CSRF Token as well. After 2 retries we'll be sent back to `login` where we need to enter password again.

![picture 81](/assets/images/26f07ba2892357b351070b45d7353f0a028a97e8fe92859e13800c15638b82e7.png)  

In order bypass CSRF and auto-logout we need to automate the following:
1. Get Request to `/login`. Capture the CSRF here
2. POST Request to `/login`
3. GET /login2
4. POST `/login2` with CSRF.

![picture 82](/assets/images/e3a7ecadcebb7d311efda30c7f2f3855c32bc107caac384de0e7a90faf1dc5c7.png)  

## 2FA Exploitation

We can use Project Sessions with Macro that would help us retrieve new CSRF token. As already mentioned, this would be the requests that we need:

![picture 83](/assets/images/1910a427c191bc481c3e20553845340ff7bf951155cd670cee54dafaec27ee39.png)  

If we do a test run, we can see that we're asked for 4-digit code and CSRF token seems to be present as hidden field.

![picture 84](/assets/images/a6e26b55747e0c0c9a820b476b74f3066d2fac2d6bb6678bb13376f15bc45d8f.png)  

Macro has now been set up:

![picture 85](/assets/images/c0a21a30cac15f7b9d9eeae521671f5a84c75b33eec9ab140ad64bc39f4f0285.png)  

Rule Settings were changed as i'll be using Target Scope AND `Extender` for `Turbo Intruder`
![picture 86](/assets/images/7ff0d458a85b4b2df30b69e38e4b8554cf52ab9ec77ba2e774d2863ad47f1457.png)  

Now if we start Intruder, CSRF token should get changed automatically so We don't need to worry about CSRF and/or do anything with it.

![picture 88](/assets/images/76abb3871cda02594cf7d44a1df2992c649ed9109411c7e5e33bd3b3ca630d21.png)  

Payload was set as following using `Numbers`:

![picture 89](/assets/images/e3a0bac8dc7824c0a077fe4a277525c35b7422dfa7d47a9ba72659bf50a811ab.png)  

Remember, we would see HTTP Error 400 if CSRF token would not match, so brute-force works!

![picture 87](/assets/images/878817350a3ab6a03b18bf6cbd9215af13697c33ec19d147fecced9454531e59.png)  

This would also work with `Turbo Intruder`. I've changed script as following:

```py
# Find more example scripts at https://github.com/PortSwigger/turbo-intruder/blob/master/resources/examples/default.py
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           requestsPerConnection=1,
                           #pipeline=False,
                           engine=Engine.BURP
                           )
    for i in range(9999):
        engine.queue(target.req, '{:d}'.format(i).zfill(4))
        

@MatchStatus(302)
def handleResponse(req, interesting):
    if interesting:
        table.add(req)

```

There's a PIN

![picture 90](/assets/images/21bc0d3a8ee36cb7aedf435d9ca04973da6b2b07e5158f414076a0c8e554af70.png)

I haven't thought of that, but attack should've been stopped as soon as `HTTP 302` was returned.

Anyways, the lab has been solved!

![picture 91](/assets/images/d0d86b459012cc3e0d2e1dc25c7a0e2c2541ada7828b2391c9b986b21369bb61.png)  
