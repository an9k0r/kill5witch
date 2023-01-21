---
title: (Portswigger/WebAcademy) - crAPI Writeup
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-12-21 09:00:00 +0200
categories: [Web Application, API]
tags: [Notes, Web Application, API]
math: true
mermaid: true
image:
  src: 
  width: 694
  height: 515
  alt: image alternative text
---
# Intro
This post is all about the vapi which is hosted on [github](https://github.com/roottusk/vapi) and it has been created by [Tushar Kulkarni](https://twitter.com/vk_tushar). 

---
**The Plan**

This vulnerable API has no Frontend like e.g., [craPI](https://github.com/OWASP/crAPI), so i'll just take [documentation](https://www.postman.com/roottusk/workspace/vapi/overview) and import it to my Postman Instance as a collection. 

I'll be using Kali Linux and following tools:
- Burp Suite Community
- Postman
- ffuf
- wfuzz
- jwt_token

## TOC
- [Intro](#intro)
  - [TOC](#toc)
- [Vulnerabilities](#vulnerabilities)
  - [API1:2019 Broken Object Level Authorization\<](#api12019-broken-object-level-authorization)
  - [API2:2019 Broken User Authentication](#api22019-broken-user-authentication)
  - [API3:2019 Excessive Data Exposure](#api32019-excessive-data-exposure)
  - [API4:2019 Lack of Resources \& Rate Limiting](#api42019-lack-of-resources--rate-limiting)
  - [API5:2019 Broken Function Level Authorization](#api52019-broken-function-level-authorization)
  - [API6:2019 Mass Assignment](#api62019-mass-assignment)
  - [API7:2019 Security Misconfiguration](#api72019-security-misconfiguration)
  - [API8:2019 Injection](#api82019-injection)
  - [API9:2019 Improper Assets Management](#api92019-improper-assets-management)
  - [API10:2019 Insufficient Logging \& Monitoring](#api102019-insufficient-logging--monitoring)
  - [ARENA - Server-Side Request Forgery](#arena---server-side-request-forgery)


# Vulnerabilities

After having imported API documentation into Postman, we'll notice that documentation is very conviniently arranged.

![picture 1](/assets/images/de6f165b40ec424c99a715e530821576aa2bef5783332a0d210a26205841fa60.png)  

Each API folder is for each API vulnerability type. We know what we have to do, so let's begin...

## API1:2019 Broken Object Level Authorization< 

> APIs tend to expose endpoints that handle object identifiers, creating a wide attack surface Level Access Control issue. Object level authorization checks should be considered in every function that accesses a data source using an input from the user.
>
> Source: https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa1-broken-object-level-authorization.md

In order to even send a request in postman, we need to set variables. To get started we need `host` variable. I've added `vapi.local` to my `/etc/hosts` file, just to make the requests more distingishable.

![picture 3](/assets/images/8049c2c6560bd8cde77af7200597a990fd80badf886f660f9f500d7fe1bcbe76.png)  

Now let's go to `API1` and use POST `Create User` to create a user:

![picture 4](/assets/images/cf392fa8505aea16bb031bea2da612efbba1b42d5d625db31e41ca4c85d58e67.png)  

I've got an ID of `5`. This should be added to `api1_id` environment variable, same as `api1_auth`. Otherwise check Console!

If we issue GET `Get User` request, we should see our data

![picture 5](/assets/images/24b845ab9e6a3af546e2d1a49f11d3d78cb5936458948cd6db6bc36202a5bc96.png)  

If we change ID to `1`, we can read other users information, leading to BOLA vulnerability

```
{
    "id": 1,
    "username": "michaels",
    "name": "Michael Scott",
    "course": "flag{api1_d0cd9be2324cc237235b}"
}
```

Same happens with PUT method if we set an ID to some other users, like `2`.

![picture 6](/assets/images/d9405e25732f6ae40ad6e15a5be4c74cbda852a8216f97eda12a5861d82380e2.png)  

We have now overwritten user that was stored under ID of `2`.

## API2:2019 Broken User Authentication

> Authentication mechanisms are often implemented incorrectly, allowing attackers to compromise authentication tokens or to exploit implementation flaws to assume other user’s identities temporarily or permanently. Compromising a system’s ability to identify the client/user, compromises API security overall.
>
> Source: https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa2-broken-user-authentication.md

In this section, we have to check `Resources` folder to get list of credentials:

![picture 7](/assets/images/bab483a04f3e46429b23897d1874ae7b8afd523778acc9e15c904e46c7bcccc7.png)  

This are credentials provided

```
 luka@yokai  ~/dckr/vapi/Resources/API2_CredentialStuffing   master  head -n 5 creds.csv 

brown.grimes@hotmail.com,w_5yhfEN
reuben.heaney@hotmail.com,8JhcB_mH
dcronin@robel.com,V$qe{8+3
hcollier@veum.com,vVsU7/yN
vemard@gmail.com,gRfJ3$U7
```

There are 2 endpoints present:

![picture 8](/assets/images/fc53c4d85fea5306d4a6f90c58dd74ba24e4ca7cd32fad4370ea87dd82ca7ee4.png)  

I've set a proxy to Burp and sent a request to brute-force the credentials provided!

I've splited the `creds.csv` into 2 separate files

```
# Users
cat creds.csv | awk -F"," '{print $1}' > users
# Passwords
cat creds.csv | awk -F"," '{print $1}' > passes
```

In Burp, we'll use `Pitchfork` mode.

Alternatively, we could process single file twice in Burp, usernames before the comma and passwords after the comma.

We can brute-force credentials using FFUF as well

```
ffuf -w users:USERS -w passes:PASS -u http://vapi.local/vapi/api2/user/login -d '{"email":"USERS","password":"PASS"}' -H "Content-Type: application/json" -mode pitchfork -mc all -fc 401
```

FFUF ended up with 3 working credentials:

```
[Status: 200, Size: 89, Words: 1, Lines: 1, Duration: 1769ms]
    * USERS: savanna48@ortiz.com
    * PASS: zTyBwV/9

[Status: 200, Size: 89, Words: 1, Lines: 1, Duration: 1971ms]
    * USERS: hauck.aletha@yahoo.com
    * PASS: kU-wDE7r

[Status: 200, Size: 89, Words: 1, Lines: 1, Duration: 1829ms]
    * USERS: harber.leif@beatty.info
    * PASS: kU-wDE7r
```

Authentication should work with one of the credentials and we should get a token back.

![picture 9](/assets/images/be57f5b575d639e16c1b844c081811c263f8f9648a2b9817cffa7105af039031.png)  

We can also see that we have `Rate Limiting` problem here.

Using `/vapi/api2/user/details` (GET) we'll notice that we get information about every user out there.

![picture 10](/assets/images/2c30483affd188c333bd8bb0b883bd00f66b7c0c73d7efacfe406f0644a0b9bb.png)  


## API3:2019 Excessive Data Exposure

> Looking forward to generic implementations, developers tend to expose all object properties without considering their individual sensitivity, relying on clients to perform the data filtering before displaying it to the user.
>
> Source: https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa3-excessive-data-exposure.md

API3 only has only one Endpoint in documentation

![picture 12](/assets/images/281cf158c70a36d74c1289fe1c3d4503a0e4f78e509e74e36dc30cb7d92303ce.png)

We can create a new user but that's it.

For `API3` challenge we'll need to check Resources that come with the app.

We'll get an `.apk` which we'll have to reverse. I've used `jadx-gui` for that!

![picture 13](/assets/images/f2bbe955f62bfa712248701f97d5a38200446a317e8b57927a2732d15c14348a.png)  

See the endpoint there?

What if we send a get request to that endpoint?

![picture 11](/assets/images/c47cefce7c8e430de579dfb0c88d5793ce10fbe5ae8a93a8b21e21bc87a89690.png)  

Yes we get a flag and as we can notice, much more information that's necessary. 

## API4:2019 Lack of Resources & Rate Limiting

> Quite often, APIs do not impose any restrictions on the size or number of resources that can be requested by the client/user. Not only can this impact the API server performance, leading to Denial of Service (DoS), but also leaves the door open to authentication flaws such as brute force.
>
> Source: https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa4-lack-of-resources-and-rate-limiting.md

The `API4` has 3 documented Endpoints

![picture 14](/assets/images/8c47514f7ba5bb8764bc28f9617a09ff99e0bc847121569fb3df034242ed74a4.png)  

We can use `Mobile Login`, however this is only 1 part of the login process. We need to provide OTP as well and here is where vulnerability lies. `Verify OTP` has no rate limiting set, making it easy for us to brute-force the OTP code. We can use Intrudr or WFUZZ

```
wfuzz -z range,0000-9999 -u http://vapi.local/vapi/api4/otp/verify -d '{"otp":"FUZZ"}' -H "Content-Type: application/json" --hc 403
```

If we provide the right OTP, `key` will be returned and environment variable set, so we can retrieve the flag in `Get Details`

## API5:2019 Broken Function Level Authorization

> Complex access control policies with different hierarchies, groups, and roles, and an unclear separation between administrative and regular functions, tend to lead to authorization flaws. By exploiting these issues, attackers gain access to other users’ resources and/or administrative functions.
>
> Source: https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa5-broken-function-level-authorization.md

We have to documented endpoints here for `API5`. We can create user...

![picture 17](/assets/images/6bd8e4c51546761f341c5c34d9db2cbd438411a3b900a45c95cf76eea40a1898.png)  

... and we can retrieve its information. 

![picture 16](/assets/images/aca17c1d710a62a19119a6dae02b28200d8f6da7208fe7c23c2c56d4c71e9c8d.png)  

There is no BOLA, so we cannot ask information for other user by changing its ID.

We can however try to send request to different path that is not documented and sounds like it could be right ==> `/vapi/api5/users`.

![picture 15](/assets/images/c7719b9409bd12027d63fd1e78630c813f15f650d744edd750a2a1f09fd8fd1e.png)  

... and indeed, the endpoint does not check users credentials (or does not check authorization for it)

## API6:2019 Mass Assignment

> Binding client provided data (e.g., JSON) to data models, without proper properties filtering based on an allowlist, usually leads to Mass Assignment. Either guessing objects properties, exploring other API endpoints, reading the documentation, or providing additional object properties in request payloads, allows attackers to modify object properties they are not supposed to.
>
> Source: https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa6-mass-assignment.md

With Mass Assignment vulnerability we can change values ob some object even when we're not supposed to.

We can create user...

![picture 18](/assets/images/5442503af5b5904aba9d5baacb6cde12210c5500d92f243a3ac3c5d6b15b1f37.png)  

... and retrieve its parameters.

![picture 19](/assets/images/809a4201506a9c54b27c81227479440fb65882a371bb82d12a542d9337cf0147.png)  

Can we assign credits to ourselves when we create a new user?

Let's create a new user and add `credit` parameter with some interger to it.

![picture 20](/assets/images/84023ad5ad5df8e37e5f01aa77e79bb765a6f5c01b26427a2ed9e58e2fe1c73f.png)  

Parameter `credit` does indeed reflect on ou users object

![picture 21](/assets/images/4c756d2d37dd7669302a45cfb08d51f3b3a3d9966713c811e4b796f1ab66306d.png)  

## API7:2019 Security Misconfiguration

> Security misconfiguration is commonly a result of unsecure default configurations, incomplete or ad-hoc configurations, open cloud storage, misconfigured HTTP headers, unnecessary HTTP methods, permissive Cross-Origin resource sharing (CORS), and verbose error messages containing sensitive information.
>
> Source: https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa7-security-misconfiguration.md

For Security Misconfiguration vulnerability (API7) we have 7 documented endpoints available:

![picture 22](/assets/images/351e6d779e110e6ae0d3e611e017c8f618f184d538d6e1a799b408ba5c8bc32b.png)  

We've created user, now we should be able to log in (`API7_auth` token should be saved automatically)

![picture 23](/assets/images/be2a8f38575e5b47c3908a9275820b0444f56cfbfbdd00e774808ff8f411cae9.png)  

Perhaps irrelevant, we after login `PHPSESSID` session cookie will be set!

We can now grab a key

![picture 24](/assets/images/911f049c42b746f2f3b6343b313b7f2ff1e51c3bc2f4bed749a10295cc8425ca.png)  

Vulnerability here lies in missing CORS protection or in other words - CORS is to permisive and as such allows grabing keys from victim any location.

![picture 27](/assets/images/d977da5a86369515cc6386e1fb48a195689f2a774e82309dbf160a68315e4762.png)  

![picture 26](/assets/images/01538cff53c8d023a827343d1cf20831838e323d97018b3a2c5ccf30a02a7c96.png)  

## API8:2019 Injection

> Injection flaws, such as SQL, NoSQL, Command Injection, etc., occur when untrusted data is sent to an interpreter as part of a command or query. The attacker’s malicious data can trick the interpreter into executing unintended commands or accessing data without proper authorization.
>
> Source: https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa8-injection.md

I've sent a request to Burp and checked for SQL injection there.

![picture 28](/assets/images/44fbadb91721bf42329b11d74ca4eeeb1ea5442475e9b8c586d14be61267e406.png)  

We can use the key to get the flag:

![picture 29](/assets/images/b7cba2a34ddc7ad0b909feb7120d1b75053730cc8da107bdbe53ddc04a8a1928.png)  

## API9:2019 Improper Assets Management

> APIs tend to expose more endpoints than traditional web applications, making proper and updated documentation highly important. Proper hosts and deployed API versions inventory also play an important role to mitigate issues such as deprecated API versions and exposed debug endpoints.
>
> Source: https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa9-improper-assets-management.md

We have a single endpoint available here which is login with provided user and we're supposed to guess/brute-force PIN.

![picture 30](/assets/images/cae0e41073e87c3dd61e7e210d7d95520de690635bcb7759ab4a4f4da29e5dee.png)  

Now the problem is that we gate rate-limited after 5 attempts.

There is `v1` instance available which has no rate-limiting enabled

![picture 31](/assets/images/ed5e54d3f7e210712dd928c81538a518709ca44c2478361ce740203257faf40a.png)

Exploitation after finding the endpoint is the same as in API4.

![picture 32](/assets/images/bb9853a2793e71018568882e2394a22bd53db4ea78d5569c65db590d2bbe5c4b.png)  

## API10:2019 Insufficient Logging & Monitoring

> Insufficient logging and monitoring, coupled with missing or ineffective integration with incident response, allows attackers to further attack systems, maintain persistence, pivot to more systems to tamper with, extract, or destroy data. Most breach studies demonstrate the time to detect a breach is over 200 days, typically detected by external parties rather than internal processes or monitoring.
>
> Source: https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xaa-insufficient-logging-monitoring.md

Here we get a flag right away, but i guess we get and understand a message, right?

```json
{
    "message": "Hey! I didn't log and monitor all the requests you've been sending. That's on me...",
    "flag": "flag{api10_5db611f7c1ffd747971f}"
}
```

## ARENA - Server-Side Request Forgery

`asd` is vulnerable to Server-Side Request Forgery and vulnerability is not hard to find.

Interestingly if we try to connect with `http://localhost` or `http://127.0.0.1`, we'll get 403 response back.

![picture 1](/assets/images/716e1e6bf92ca5dc13abc66825eb9a61da1c1185e6e8293fcb8f300ab8af5e66.png)  

We can however let the server connect with external resources like `https://webhook.site` which will display that HTTP request.

![picture 2](/assets/images/6464b812f4a0606a22656ad944a7624ede749c1ceb27f5ad6c339b9b0a9cc926.png)  

Inband SSRF returns response BASE64 encoded.

We can e.g., use `file://` to read files on the system.

![picture 3](/assets/images/82f8344a647a9a4c301c17344428b0b87e6bd40374711f798031ec3639981956.png)  
