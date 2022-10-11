---
title: Business Logic Vulnerabilities
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-06 09:00:00 +0200
categories: [Web Application, Bussines Logic Vulnerabilities]
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
This post/writeup is all about the Business Logic Vulnerabilities.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/logic-flaws) Labs, but i do intent do throw other labs and writeups here as well.

Business logic vulnerabilities are flaws in the design which usually get discovered by using the web application in a way that was not intended to be used. E.g., process from checking what is in the cart to the payment.

- [Intro](#intro)
- [Excessive trust in client-side controls - define the price from the client](#excessive-trust-in-client-side-controls---define-the-price-from-the-client)
- [High-level logic vulnerability - Set product quantity in negative](#high-level-logic-vulnerability---set-product-quantity-in-negative)
- [Low-level logic flaw - Integer Overflow](#low-level-logic-flaw---integer-overflow)
  - [Solution #1 - simple Intruder and manual browser check](#solution-1---simple-intruder-and-manual-browser-check)
  - [Solution #2 - Intruder with consequent request to check total price](#solution-2---intruder-with-consequent-request-to-check-total-price)
- [Inconsistent handling of exceptional input - email address gets trimmed after 255 characters while registering](#inconsistent-handling-of-exceptional-input---email-address-gets-trimmed-after-255-characters-while-registering)
- [Inconsistent security controls - change email address to gain access to internal admin panel](#inconsistent-security-controls---change-email-address-to-gain-access-to-internal-admin-panel)
- [Weak isolation on dual-use endpoint - Removal of current-password parameters while changing password](#weak-isolation-on-dual-use-endpoint---removal-of-current-password-parameters-while-changing-password)
- [Insufficient workflow validation](#insufficient-workflow-validation)
- [Authentication bypass via flawed state machine - exploiting default behaviour](#authentication-bypass-via-flawed-state-machine---exploiting-default-behaviour)
- [Flawed enforcement of business rules - alternating coupon codes](#flawed-enforcement-of-business-rules---alternating-coupon-codes)
- [Infinite money logic flaw](#infinite-money-logic-flaw)
- [Authentication bypass via encryption oracle](#authentication-bypass-via-encryption-oracle)

# Excessive trust in client-side controls - define the price from the client

>  This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".
> 
> You can log in to your own account using the following credentials: `wiener:peter` 

This is the jacked that we're supposed to buy, i've alrady put it into the cart. Who doesn't want a "l33t" jacked, right? I've also logged in using `wiener:peter`.

![picture 161](/assets/images/812606c1b2ae000e479e32cefa62b05ed00bf92a18504c43ba83d7f910fba125.png)  

This is the request and aparently price is getting sent from the client

![picture 162](/assets/images/a3d21777032d76d8fbec79f25269f2009e89adc1b10d4911e258ae8ea9a7ed60.png)  

I'll remove the jacked from my cart and put a number `5000` in there, making it cost 50 Bucks: `productId=1&redir=PRODUCT&quantity=1&price=5000`

Now let's buy the `l33t` Jacket and solve the lab.

![picture 163](/assets/images/e7a5d882c1b2b9317025b8b535730ff94e54f6e8685eb426e2d0527232204e00.png)  

# High-level logic vulnerability - Set product quantity in negative

>  This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".
> 
> You can log in to your own account using the following credentials: `wiener:peter`

Now we have to buy the `l33t` jacket again. I've logged in and clicked on `Add to Cart`

![picture 164](/assets/images/b2b53f51782a9944e67605094f6bb91ea5625dce040a0047f419d39116199891.png)  

When we send the product to the cart, POST Request to `/cart` will be sent with following parameters/values in the body: `productId=1&redir=PRODUCT&quantity=1`.

What happens if we change `quantity` to `-1`? 

![picture 165](/assets/images/e2443b253f441a847510503a6cf450931ff6516acb408a83a2716d1cf148ecee.png)  

We should recieve some money for buying the jacket! But that doesn't happen. We recieve an error instead: `Cart total price cannot be less than zero`. Let's add the jacket and drive the total price above 0 with another product!

![picture 166](/assets/images/3a60d70bab13cdc9d032faa0cec38b92ac15c3002a18de91282f6beb9050e40e.png)  

Lab has been solved!

# Low-level logic flaw - Integer Overflow

>  This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".
> 
> You can log in to your own account using the following credentials: `wiener:peter`

Procedure is the same. Login and put the jacked in the cart.

![picture 167](/assets/images/3ca90965be63a3d9ca0da6a97d391fead77f68127d625c66c8dae29860d2dec8.png)  

Maximum amount of quantity that we can add to the cart is set to `99`.

Let's try intruder and start adding jackets to the cart. Do not put any positions:

![picture 168](/assets/images/04975ff85652f6ace911aa94ff87ca9d620858d1cdb0028916f5f25d97a2179f.png)  

Check the cart at 6040 items:

![picture 169](/assets/images/e06784b0578005b7d75d12f71ef733991e5d85465fd7b1b590bd5e42fedce153.png)  

at 17236

![picture 170](/assets/images/48e49740cc3faa0385673e479d5950db41b7ba33c3743a81390cee737edb2b9b.png)  

From here the price starts falling.

## Solution #1 - simple Intruder and manual browser check

Now we just need to make sure to stop the intruder before we hit 0 again. We wan't to end up with the Total price between `$0` and `$100.00`.

![picture 171](/assets/images/fa006ada8599099dadf5f78d50dc42ab7f225c7748a96e7cd9e25db8c32c193f.png)  

Lab has been solved.

![picture 172](/assets/images/5992c2ef3e4440e39b8ba9160bde398eff2ff8a31600a5c70e50a9f6ef84b421.png)  

## Solution #2 - Intruder with consequent request to check total price

We can also get results like this:
![picture 173](/assets/images/be893512cf60c5bdc198b3cb9b9a5df5c815eb5ac6fe2c59974b7c7a48e86f1e.png)  

To make that work i've used Project Options and added a Macro:

![picture 174](/assets/images/171b8c3b59877e5e27060456f0cf93ef431b0c01a2f10dbdff66dad91e605cc4.png)  

And everything else in Intruder Settings stays the same exept `extract grep`.

![picture 175](/assets/images/cef81c7839529887019800be67207545082db3bf9bb2394c318279aba82f5833.png)  

It's not faster or anything like that, this solution just presents the total price in the Intruder itself!

# Inconsistent handling of exceptional input - email address gets trimmed after 255 characters while registering

> This lab doesn't adequately validate user input. You can exploit a logic flaw in its account registration process to gain access to administrative functionality. To solve the lab, access the admin panel and delete Carlos. 

So it's about registering a user. This is how registration form looks like:

![picture 176](/assets/images/944703e8bc96a6944c5ebf6607fdb28a2e17dac23e895a0c77be1543218fcf5c.png)  

We also have an email client

![picture 177](/assets/images/c5c2f0dad0da1c3a62712a946f5f8892dad099b9e847a88f87a8686197fe25ec.png)  

We should use the email shown in the email client for registration!

If we register we'll recieve a Email that looks like this:

```
Hello!

Please follow the link below to confirm your email and complete registration.

https://0a3300aa04c4cf72c0d6d01200e4006e.web-security-academy.net/register?temp-registration-token=TQCEaSvNMlQZXucS22Lgj5Kkzkpv1tGf

Thanks,
Support team
```

If we try to re-register `carlos` we get following message: `An account already exists with that username`

So we've hit the bottom of this rabbit hole, so what else we have:
- During registration we'll see following message `If you work for DontWannaCry, please use your @dontwannacry.com email address`.
- If we do basic directory enumeration we'd find a `/admin` directory.

![picture 178](/assets/images/83cb82e4f7076c28531587fa2f0654159623413bf490d90f5d4b564071610bcc.png)  

If we go to that page, we'd find `Admin interface only available if logged in as a DontWannaCry user`. So we need a `DontWannaCry` user. 

This part is weird, as i'm not sure why such an Email would be sent in a first place but perhaps it has been made this way to make the lab not to hard.

If we register Email longer than 255 characters, the email will trim on the backend. 

E.g. let's use python to generate long Email address above 255 characters:

```
lukaosojnik@Yokai ~ % python3 -c "print('a'*250 + 'attacker@exploit-0a2e00bc04962b4bc025167101ea00bf.exploit-server.net')"
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaattacker@exploit-0a2e00bc04962b4bc025167101ea00bf.exploit-server.net

lukaosojnik@Yokai ~ % python3 -c "print(len('a'*250 + 'attacker@exploit-0a2e00bc04962b4bc025167101ea00bf.exploit-server.net'))" 
318
```
So our Email is now 318 characters long. If we register it, that's what's shown in the backend after logging in:

![picture 179](/assets/images/426058b3c2e06403498b693f9c719d4591b73e04d50f9ac3beeaf5e13db0d20c.png)  

Our email has got trimmed. And it's exactly 255 characters long:

```
lukaosojnik@Yokai ~ % python3 -c "print(len('aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaattac'))"
255
```

Now let's try to have it trimmed in a way so our Email ends with `@dontwannacry.com`.

Start with `attacker@dontwannacry.com` continued with our address. (**after trial an error, i've had to remove a @ from our address otherwise email would not get accepted!**)

```
lukaosojnik@Yokai ~ % python3 -c "print(len('attacker@dontwannacry.com'))"                                                                  
25
```
We're at 25 Bytes, we need to divide that from 255 and that's our filler

```
lukaosojnik@Yokai ~ % python3 -c "print(len('a'*230 + 'attacker@dontwannacry.com'))"
255

lukaosojnik@Yokai ~ % python3 -c "print('a'*230 + 'attacker@dontwannacry.com' + 'attacker@exploit-0a2e00bc04962b4bc025167101ea00bf.exploit-server.net')"
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaattacker@dontwannacry.comattacker@exploit-0a2e00bc04962b4bc025167101ea00bf.exploit-server.net
```
Now register and login!

![picture 180](/assets/images/d53382110364ee54724c772ad7c80348b2acfc44e8409335a239ba2a82600d3c.png)  

We can now go to `/admin` and delete `Carlos` and solve the lab

![picture 181](/assets/images/b2bdb806bebea0b6097de56b239875dcdb5cbebe80845a7974c42ef0045aa8f6.png)  

# Inconsistent security controls - change email address to gain access to internal admin panel
> This lab's flawed logic allows arbitrary users to access administrative functionality that should only be available to company employees. To solve the lab, access the admin panel and delete Carlos. 

This lab also involves admin panel which has been found in the previous lab `/admin`. We've used Feroxbuster to find it, but it's obviously easily guessable as well ;). If we visit it we see same message `Admin interface only available if logged in as a DontWannaCry user `.

We have `Email client` to our disposal, so let's grab the email there and register an account:

![picture 182](/assets/images/db449bbd052919b46104f58d2a003b63ed051e78dab72d730e61546536e168dd.png)  

We then recieve an Email that we have to confirm.

![picture 183](/assets/images/a1d79e22b2d407a853f41e49856e986ab0dffb0fda1b9a6be5b90179e0424554.png)  

We can change the email after login:

![picture 184](/assets/images/366cc1dad5465dbd1870c9f1883bcb76a928ba34175e99f2ed9359609e4348ca.png)  

See the `Admin panel`?

![picture 185](/assets/images/6711dd82612edba75f39415f995f1d287b24b58121455d679d31158b13139b3b.png)  

Go to `/admin` and delete `carlos` to solve the lab

![picture 186](/assets/images/393a9571aa8e02e7916e7c6f6f94eea315f8e9a555e1d19827a269247b9642dd.png)  

# Weak isolation on dual-use endpoint - Removal of current-password parameters while changing password

>  This lab makes a flawed assumption about the user's privilege level based on their input. As a result, you can exploit the logic of its account management features to gain access to arbitrary users' accounts. To solve the lab, access the administrator account and delete Carlos.
> 
> You can log in to your own account using the following credentials: `wiener:peter` 

We have a normal page where we can log in. Let's do that using provided `wiener:peter` 

![picture 187](/assets/images/7758837cbac44525aa3bacb1361b9d1e8f0317090f90762a9c47374ed5275521.png)  

This is the request that goes out if we change the password:

![picture 188](/assets/images/69fbc456a57cc66fba33a3b866518be999ecd7bcac199acadb4265a5b10a3b41.png)  

We can obviously change the password without knowing one just by removing `current-password` from the POST Body.

![picture 189](/assets/images/ca71a4e55580c5331b820832ac2508f4e641e84168ce5da4df2d244d27f84c7e.png)  

And this works for `administrator` as well.

![picture 190](/assets/images/aeaf6b122c50eb4183678bd7e47c2becf443faa477845f3093fd113a9be5950d.png)  

Log in as `administrator` go to `Admin panel` and delete `carlos` in order to solve the lab!

![picture 191](/assets/images/386c8d8fecf326d65b8b241ac70ae9ba2e1ced1975dafa602fc3efe16a47841e.png)  

# Insufficient workflow validation
>  This lab makes flawed assumptions about the sequence of events in the purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".
> 
> You can log in to your own account using the following credentials: `wiener:peter` 

This lab is about buying the `l33t` leather jacket again. Let's try to do that, but login first!

After having done all that, this is the cart:

![picture 193](/assets/images/ccbd6829e20f208b6ca9d1f162845a90120f236d0859da13177d3f1e620e902b.png)  

We cannot afford it as `Not enough store credit for this purchase`

This is the request in Burp. Notice the consquent `/cart?err=INSUFFICIENT_FUNDS`.

![picture 194](/assets/images/ab4a5f3ca53ce22fcf290f2fac91e7aee73649fb70d86ed5b8ad5db145fc1433.png)  

If we buy a product and we do have sufficient funds, the next consequent request would be `/cart/order-confirmation?order-confirmation=true`

If we send this GET Request to `/cart/order-confirmation?order-confirmation=true` our order would be completed and lab would be solved!

![picture 195](/assets/images/dc917113dcf26c6c25bfc507a42899ecad34db9c874ba49b4395a53f2d427b62.png)  

# Authentication bypass via flawed state machine - exploiting default behaviour

>  This lab makes flawed assumptions about the sequence of events in the login process. To solve the lab, exploit this flaw to bypass the lab's authentication, access the admin interface, and delete Carlos.
> 
> You can log in to your own account using the following credentials: wiener:peter 

There's not many options what to do here. Let us login

![picture 196](/assets/images/58a10adc463f4af9e8b10d9019dd0585422d090fc1580edc9db41299bb156644.png)

After logging in we can select a role. We can choose from `User` and `Content creator`. 

![picture 197](/assets/images/bd630a902376ccbe887b1995cff929fd5aa8123d27ff4e6f88305af91e44ce35.png)  

If we choose any one of them we cannot get administrator privileges, even if we use `role=admininstrator` or `role=admin` as parameter. What is default behaviour anyways. What happens if we intercept and drop the request after login?

![picture 199](/assets/images/1164192269d58670262d27e8877a4486d3fcf3af7c5be88e0408b1f81bf017ab.png)  

We can open `/admin` and see that our role has defaulted to `adminstrator`

![picture 200](/assets/images/e847693121e5b87b25e2aa032effbb413c5d1846342c22ecd2c4012ca7f89126.png)  

Delete `carlos` to solve the lab.

# Flawed enforcement of business rules - alternating coupon codes

>  This lab has a logic flaw in its purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".
> 
> You can log in to your own account using the following credentials: `wiener:peter` 

There we are buying the `l33t` jacket all over again. This is the website when the lab starts:

![picture 201](/assets/images/d1e396b83ad276c55d259c839db04e18aed033bbe614a9c26393e916917cd572.png)  

There's a coupon code that we can use. Let's put the jacket to the cart and copy the coupon code.

![picture 202](/assets/images/a1554f6b35cc45525db427c8513ad7d97390f143ad34e5daa348535b7e66dfa6.png)  

Heck, we just get a 5$ reduction and we cannot re-apply the coupon as we get `Coupon already applied` message. 

If we scroll to the bottom in the shop, we can sign up for newsletter

![picture 203](/assets/images/f66bdebd518df5d16d8aeeafb7999a87775efc52338b191fd9a4e9a9a4e6a2c3.png).

We get an alert with a new coupon code: `SIGNUP30`:

![picture 204](/assets/images/06ac3dbdc31bf33e95f1fe54b729e7d605d58196e1b64464dd4a31b3eea6d20d.png)  

We can apply that coupon to.

![picture 205](/assets/images/2ea915ebe7e7d4ed08720129d61ee3e097025568755b4ee4bcc21058167088d4.png)  

We cannot apply two same coupons at one, we can however alternate them, buy the jacked and solve the lab!

![picture 206](/assets/images/97044c2a4ad3bb780c397c1ccfd08e60514b3b7d7e06d5e343ee43f1aff5e502.png)  


# Infinite money logic flaw
>  This lab has a logic flaw in its purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".
> 
> You can log in to your own account using the following credentials: `wiener:peter` 

If we login using provided credentials we see that we can redeem gift cards

![picture 207](/assets/images/1a160aa9462fe9923bd21af42376bbe96ab2fc4eeb77944151706ff40f43c153.png)  

We also have an `Email client` and there's `newsletter sign-up`. Let's copy our email adress `wiener@exploit-0a72005b03bfa152c1f90627018c00df.exploit-server.net` and use it for newsletter.

We get an alert (not sure if we need an email - probably not) 

![picture 208](/assets/images/8f164491ea536e84274a0badd20712c16192a35b580ef9299451a3b9f411b442.png)  

We should not the coupon code `SIGNUP30`.

If we check the shop we can buy Gift Cards for 10$.

![picture 209](/assets/images/46e19917d3e84b6b5f41b3c242d5d0c278d91f4fa628f68590beef15158c9918.png)  

Now,.. we can buy gift card for 10$ and use coupon code to reduce 30% (3$) and redeem it again.

![picture 210](/assets/images/1f0aa2e716516afbe62ed7195cf98c8fce102ed2c49214d6f3041d53ecb4ce80.png)  

There's code that we need to redeem:

![picture 211](/assets/images/67f7d8148301c0b410729c867fa5a270bf482501534208539c6c4f2966e01102.png)  

... and there we see that we've earned 3$.

![picture 212](/assets/images/217d86a4aa15abf5826662055893c0beb5c41d2967ba8571877ddfb1e0bf73fe.png)  

Now we have to automate this using Macros as there's CSRF Token that is being sent along!

![picture 213](/assets/images/08d1e1726ca3211d4017a395ac518b0537f913efa497b8f72d8e9de272a6dfd3.png)  

We basically need all POST Requests from sending Giftcard to the card and applying the `gift-card`.

We can now do test run

![picture 214](/assets/images/b98d6b2460bad6d57e899e5d1ce917b6573c30fb2855b66a94b7441ad3c1cef9.png)  

It has worked until buying the card, however gift-card value has to be changed dynamicaly as i'm seeing 400 `Invalid gift card`. I will apply the bought `gift-card` manually and check the settings for 5th request.

The 5th request should now be using `Derive from previous response`. (Response 4)

![picture 218](/assets/images/bb383a868faf8c8ce58069013c9ac01fdd1b53d34031f62b29a148ca3e885363.png)  

In 4th Request we will need to define new `gift-card` parameter so it will be used in the last 5th request:

![picture 219](/assets/images/e8b03c1b63079d507ce92b86d6cace0d3492f55e855e887e4449f20f0d870756.png)  

If everything went well we should see 302 status after 5th request and 3$ should have been added to our account.

![picture 217](/assets/images/7840580434ab071e19652983118b38fc95793b39442957202bc5ccbff75e2766.png)  

Now it's time to send this to intruder or Turbo Intruder.

Make sure to define scope. I'll use `Turbo Intruder` so i've chosen Extender only.

![picture 220](/assets/images/8a4f1a8a85b9d05b7360033abf0c9e0f5b37f93b25b6bd3f47681e35506f05fb.png)  

I've used simple script which just issues 200 requests:

```py
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           requestsPerConnection=1,
                           pipeline=False,
                           engine=Engine.BURP
                           )
    i = 0
    while i < 400:
        engine.queue(target.req)
        i = i + 1


def handleResponse(req, interesting):
    # currently available attributes are req.status, req.wordcount, req.length and req.response
    if req.status != 404:
        table.add(req)

```

The funds should go up with every request:

![picture 221](/assets/images/fd7dd00d24171ce8d54d9844e707e48a71617e1608c3a2ab740df915bca2c513.png)  

With sufficent funds we can buy the `l33t` jacket and solve the lab.

![picture 222](/assets/images/6471777635e9ae2509f9cbfc35c06706962e304a071555ca099877b7f433cd6b.png)  

# Authentication bypass via encryption oracle
>  This lab contains a logic flaw that exposes an encryption oracle to users. To solve the lab, exploit this flaw to gain access to the admin panel and delete Carlos.
> 
> You can log in to your own account using the following credentials: `wiener:peter`

