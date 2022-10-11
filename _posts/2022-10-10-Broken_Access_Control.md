---
title: Broken Access Control
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-08 09:00:00 +0200
categories: [Web Application, Broken Access Control]
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
This post/writeup is all about the Broken access control.

I'll be using primarily [Portswigger Web Academy](https://portswigger.net/web-security/access-control) Labs, but i do intent do throw other labs and writeups here as well.

Information disclosure is all about disclosing information that was not intended to be exposed to user, like debug page for example or leaks data that is intended for privileged or other users.

## TOC

- [Intro](#intro)
  - [TOC](#toc)
- [Theory](#theory)
- [Unprotected admin functionality](#unprotected-admin-functionality)
- [Unprotected admin functionality with unpredictable URL](#unprotected-admin-functionality-with-unpredictable-url)
- [User role controlled by request parameter](#user-role-controlled-by-request-parameter)
- [User role can be modified in user profile](#user-role-can-be-modified-in-user-profile)
- [URL-based access control can be circumvented](#url-based-access-control-can-be-circumvented)
- [Method-based access control can be circumvented](#method-based-access-control-can-be-circumvented)
- [User ID controlled by request parameter](#user-id-controlled-by-request-parameter)
- [User ID controlled by request parameter, with unpredictable user IDs](#user-id-controlled-by-request-parameter-with-unpredictable-user-ids)
- [User ID controlled by request parameter with data leakage in redirect](#user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)
- [User ID controlled by request parameter with password disclosure](#user-id-controlled-by-request-parameter-with-password-disclosure)
- [Insecure direct object references](#insecure-direct-object-references)
- [Multi-step process with no access control on one step](#multi-step-process-with-no-access-control-on-one-step)
- [Referer-based access control](#referer-based-access-control)

# Theory

Access control (or authorization) is the application of constraints on who (or what) can perform attempted actions or access resources that they have requested. In the context of web applications, access control is dependent on authentication and session management:

- Authentication identifies the user and confirms that they are who they say they are.
- Session management identifies which subsequent HTTP requests are being made by that same user.
- Access control determines whether the user is allowed to carry out the action that they are attempting to perform.

# Unprotected admin functionality
>  This lab has an unprotected admin panel.
> 
> Solve the lab by deleting the user carlos. 

If we check the Site map we can find `/administrator-panel` which is present in `robots.txt`

![picture 1](/assets/images/51a1d5d2ee6cc2824c90441d054c2e3b4d9191caa85829364e30ba87977c2845.png)  

We can simply open the panel and delete the `carlos` and solve the lab

![picture 2](/assets/images/43237d9ed1850aee13c797c78cb0c4d36cd73f3248edc439e34c5662347afe26.png)  

# Unprotected admin functionality with unpredictable URL

>  This lab has an unprotected admin panel. It's located at an unpredictable location, but the location is disclosed somewhere in the application.
> 
> Solve the lab by accessing the admin panel, and using it to delete the user carlos

This lab is very similar to the previous one, we however need to check the source to find the `Admin panel`.

![picture 4](/assets/images/7d6f0796e20b7e75b116e99a5e52b7c2c74a567a5681b2a555a933ddec79da3a.png)  

When we access the `Admin panel`, we just have to delete `carlos` user to solve the lab.

![picture 5](/assets/images/582fb9a13c11d1f57d0450775f5328b93b139b842180b4052f9f2508064cd996.png)  

Alternatively we could have a look in `LinkFinder`.

# User role controlled by request parameter
>  This lab has an admin panel at /admin, which identifies administrators using a forgeable cookie.
> 
> Solve the lab by accessing the admin panel and using it to delete the user carlos.
> 
> You can log in to your own account using the following credentials: wiener:peter 

Let's login using provided credentials `wiener:carlos`:

![picture 6](/assets/images/0e60237a8514fa936f798cd426911ca3d79017b1f4762f7d7e14d3b43a66bfda.png)  

If we check the request we can notice that administrator's role is being controlled by cookie.

![picture 7](/assets/images/42af96e60d76ed2085951c64156f9a44cd5203248206740c1e035578a07b5b06.png)  

We can change that cookie in the browser directly, or use `Match and replace`

![picture 8](/assets/images/f55e105c10ecff05793b91f49322374c836ab0071c364e0836907c02dfc241fd.png)  

Either way, go to `Admin panel` and delete `carlos` when done to solve the lab

![picture 9](/assets/images/1bac2229f6de0f0f93511240064bddd3d5037def47fe62808a3e3c5a613b5c63.png)  

# User role can be modified in user profile

>  This lab has an admin panel at /admin. It's only accessible to logged-in users with a roleid of 2.
> 
> Solve the lab by accessing the admin panel and using it to delete the user carlos.
> 
> You can log in to your own account using the following credentials: `wiener:peter`

First we have to login using provided credentials `wiener:peter`.
If we make a change to profile - we can change Email address, we would notice `roleid` parameter in the request

![picture 10](/assets/images/83e410d8fadc69704fcd2f21aee6b0681d4afc5d1d1dd754da7f961bf9a555ca.png)  

We can modify that parameter:

![picture 11](/assets/images/eaa5cb7c33a8dd3e6308eed4356136141ede735b50cdee61d67927044a6dcc83.png)  

We can then go ahead and solve the lab by deleting `carlos` from `Admin panel`

![picture 12](/assets/images/cfba5b7a7b57777c619f6358da51f60d932ce4ebc3ddbcf2c110b8183e61a179.png)  

# URL-based access control can be circumvented

>  This website has an unauthenticated admin panel at /admin, but a front-end system has been configured to block external access to that path. However, the back-end application is built on a framework that supports the X-Original-URL header.
> 
> To solve the lab, access the admin panel and delete the user `carlos`. 

Upon starting the lab, let us go to `Admin panel`:

![picture 13](/assets/images/100260e8d5dac2683d5f725c625b36a9dd6f426037248ec641aaf669bd9f63d1.png)  

Disclaimer: We get `"Access denied"`.

Now at this point it was not clear to me what to do, as if i just go and add `X-Original-URL` header, we still end up with `Access denied` and cannot tell that Header is being used.

If we do that in root however `/` and place invalid header like `/invalid` (taken from Solution!), we get `404 Not Found` just because of `X-Original-URL` header.

Changing `X-Original-URL` value to `/admin`, leads us unauthenticated (!) directly to the `Admin panel`.

![picture 14](/assets/images/4981814e003633ef00917c1c8915ff1e1ae81e71a2b0bb707bef2c5c4300f1fd.png)  

We now have to change the GET request to `/?username=carlos` and modify the header `X-Original-URL: /admin/delete`

![picture 15](/assets/images/62290795a19eb12d6d41d0767aba9bf655b569cc449199879fd47ab6f38b5f27.png)  

PS: `Match and replace` did not really work as then CSS,JS wasn't loaded etc.

# Method-based access control can be circumvented
> This lab implements [access controls](https://portswigger.net/web-security/access-control) based partly on the HTTP method of requests. You can familiarize yourself with the admin panel by logging in using the credentials `administrator:admin`.
> 
> To solve the lab, log in using the credentials wiener:peter and exploit the flawed access controls to promote yourself to become an administrator. 

As we were given this possibility we can login using provided `administrator:admin` credentials and promote `administrator` to admin again and check the request in burp to see what's actualy going on.

![picture 16](/assets/images/ab3496ed5f6eb9707dcb2da47f2e95dd3b83fa683184bbb059a4de008daaebc9.png)  

We cannot do it, but we've captured the request!

![picture 17](/assets/images/0c53a4e518b0a20977cff6c5f4dfb0282aa024d662ce05ee51a1221d00d06591.png)  

Now login as `wiener:peter` and send a request to `/admin-roles`:

![picture 18](/assets/images/79b6641ab95f485019c69c1f5f49fb265c1c2a039de81d1cf51688fdeaa43537.png)  

`Unauthorized`. How about GET request?

![picture 19](/assets/images/dd8fda20f41ed0eecc9eb91aa3f9f852e7f80c865165aea186e374af8e86a9b9.png)  

Now we see the message that `username` parameter is missing. 

Can we upgrade ourselves (`wiener` = normal user) to administrator?

![picture 20](/assets/images/779cdb80d112f48d02abaf5aa583438a017d658eaf91e5276cdec80f379f56d4.png)  

Sure we do & another lab has been solved!

# User ID controlled by request parameter 
>  This lab has a horizontal privilege escalation vulnerability on the user account page.
> 
> To solve the lab, obtain the API key for the user `carlos` and submit it as the solution.
> 
> You can log in to your own account using the following credentials: `wiener:peter`

When we login we get an API key assigned!

![picture 21](/assets/images/b97f9bd94d886c17b5ec865f7ece2b6ec589b58c35a670c2c57970fce4ac1d8a.png)

If we click on `My account` again, another request would be issued

![picture 22](/assets/images/bc38786fede1dabcf5c979b3eaa9e8512482f9619c8579acaaaf1b6de3033e13.png)  

We can request for API from `carlos`

![picture 23](/assets/images/6703eaeaa264a2f9dc7c5b9c9858db04dd1fa8faa95816ae267d7664d2fd2383.png)  

Submit the API Key and solve the lab!

# User ID controlled by request parameter, with unpredictable user IDs 

> This lab has a horizontal privilege escalation vulnerability on the user account page, but identifies users with GUIDs.
> 
> To solve the lab, find the GUID for `carlos`, then submit his API key as the solution.
> 
> You can log in to your own account using the following credentials: `wiener:peter`

If we login using `wiener:peter` credentials that were provided by us and check the `My account` link, we'll see that it's pointing to GUID from `wiener`. 

![picture 24](/assets/images/88e71316513cd2542fb0b52eeca7aeb2c0523d666d4b69b868c26c3ee3b5fb69.png)  

We would now need to find the place where we could display the GUID of `carlos`!

We can see that `carlos` has dropped few posts:

![picture 25](/assets/images/2f7aa894afc0b8dca69cf722fb8bc9645403d4ae3135aff429d15d7762f8380a.png)  

If we check the source we'd find the `carlos`'s GUID
```html
<a href='/blogs?userId=a6e6c070-4117-4627-af43-e299301ae904'>carlos</a></span> | 12 September 2022</p>
```

We can use it to display the API key:

![picture 26](/assets/images/4521b31ae3807d1520828fb4463407b93e37dacb894a460563d0d67f0c208c2c.png)  

# User ID controlled by request parameter with data leakage in redirect 

>  This lab contains an access control vulnerability where sensitive information is leaked in the body of a redirect response.
> 
> To solve the lab, obtain the API key for the user carlos and submit it as the solution.
> 
> You can log in to your own account using the following credentials: `wiener:peter` 

Let's login using provided credentials `wiener:peter`

![picture 27](/assets/images/88215b0c0b15f32682c75709fc958f49baad54c73cedd14266a2f4707871248f.png)  

If we again click on `My account` the id should be set.

![picture 28](/assets/images/fa5765ef5989e51d6e71c47ce9c808b93e86635325f3b448e61daf5ce9c9a0b3.png)  

Now let's swap it with `carlos` and check the Burp.

We can notice the `302` redirect however page will still load and disclose the API of `carlos`'s

![picture 29](/assets/images/bbc5da93d013d5e9ddd74c93c59b6520b31b6c79a8bb9f39ccb10b2e81a21721.png)  

Submit the API key and solve the lab!

# User ID controlled by request parameter with password disclosure

>  This lab has user account page that contains the current user's existing password, prefilled in a masked input.
> 
> To solve the lab, retrieve the administrator's password, then use it to delete carlos.
> 
> You can log in to your own account using the following credentials: `wiener:peter`

Let's login using provided credentials `wiener:peter`. If we click on `My account` we would notice that password get's loaded to the frontend which is weird.

![picture 30](/assets/images/e7fea280e185445f24406b38670dd832950e74c0335ae3e636bb11e47a342bc1.png)  

Now, we can also achieve that for `administrator`.

![picture 31](/assets/images/705c1698e0950aae901ae67e8affb1cf7bb9b7d7f516734f17a2619c7d1af57d.png)  

In order to solve the lab, we need to login using password found and login as `administrator` and delete `carlos`.

![picture 32](/assets/images/3b1127e703962c24a17a89bc457029cc0a3aa25cde6a6dc243643acdf075a633.png)  

# Insecure direct object references
>  This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.
> 
> Solve the lab by finding the password for the user `carlos`, and logging into their account. 

We can make use of live chat in this lab.

![picture 33](/assets/images/47c53cfd0dbe4c61c87062028a5a42a59e7cff3d5e7aa70fbe02305801f07c97.png)

Live chat has `Download transcript` option and if we use it, we can see our chat history.

![picture 34](/assets/images/315540709f9923e67dcf60fffd6f6f5d7e9d5f25d7c14408d672bf45de0b2b59.png)  

What happens if we swap the id of `2` with `1`?

![picture 35](/assets/images/8a46b2651372e0551e186ecb5d8d74631c33f8df837f88b49dee4999df320c0f.png)  

Yeah, we can see the chat that belongs to other user along with the `carlos`s password which we can use to login and solve the lab.

# Multi-step process with no access control on one step 

>  This lab has an admin panel with a flawed multi-step process for changing a user's role. You can familiarize yourself with the admin panel by logging in using the credentials `administrator:admin`.
> 
> To solve the lab, log in using the credentials `wiener:peter` and exploit the flawed access controls to promote yourself to become an `administrator`. 

Let's login using `administrator:admin` that was provided to us.

User Promotion is done in 2 steps:

**FIRST REQUEST**
![picture 36](/assets/images/367fe34ec69f495a341ffc9ca4f3ad4ae6837c69e8e72277be322467ed6a74e9.png)  

POST Request
```
POST /admin-roles HTTP/1.1
Host: 0afd004904aed73fc06eab33000b00a8.web-security-academy.net
Cookie: session=0C1CqV6e1kBdfkaON0eKHjwamc4GDKrO
...
Connection: close

username=administrator&action=upgrade
```

**SECOND REQUEST**
![picture 37](/assets/images/4f74f8af6cc1b3f1756d0886c731fd3b50fb06f85e022e93197cc81d31b3f46d.png)  

POST Request
```
POST /admin-roles HTTP/1.1
Host: 0afd004904aed73fc06eab33000b00a8.web-security-academy.net
Cookie: session=0C1CqV6e1kBdfkaON0eKHjwamc4GDKrO
...
Connection: close

action=upgrade&confirmed=true&username=administrator
```

Now let's try replay same requests using `wiener` user.

Replaying first request ends with `401 Unauthorized`:

![picture 38](/assets/images/a2581e0f0aeb5e87d64ff5f2d8c858bf1f47bd8d01a8d49fd1ea536f09cfd9a6.png)  

Replaying second request seems to work! :)

![picture 39](/assets/images/211edf0c7de8237bb0a416a82fb5078f878635ee7566006917262d1451343f1a.png)  

To solve the lab, simply promote `wiener` like seen in the screenshot above.

# Referer-based access control 

>  This lab controls access to certain admin functionality based on the Referer header. You can familiarize yourself with the admin panel by logging in using the credentials administrator:admin.
> 
> To solve the lab, log in using the credentials wiener:peter and exploit the flawed access controls to promote yourself to become an administrator. 

Let's login using provided `administrator:admin` and try to promote `carlos` user. 

![picture 40](/assets/images/8fa10d1511ff4ad5c235fa44d8f5859b77c7bb99c7f30376b7b4cdeadbe1c1ee.png)  

Send the query above to `Burp Repeater`

Let's login using`wiener:peter` and try to use the `wiener`'s session. It will work if `Referer` header is set to `hostname/admin`.

![picture 42](/assets/images/8752f06445ed0ce06f2a11fc7cb640512b641548489ead52c64d1531a272b06c.png)  

Lab has been solved.

![picture 41](/assets/images/5b8e3291e774c769c22892099401ff09b2477b4d4443490f2603d89512c2fe84.png)  

