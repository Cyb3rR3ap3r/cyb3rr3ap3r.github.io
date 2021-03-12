---
layout: page
title: Grandpa - HackTheBox
permalink: /posts/ctf/hackthebox/grandpa.html
---

# Grandpa - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.13`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3a50e2ff-e137-40ba-999f-9837c05fb549/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3a50e2ff-e137-40ba-999f-9837c05fb549/Untitled.png)

Looks like we only have port 80 to deal with, lets check it out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/94d8f162-daf4-46a1-b8fe-c4889347daa4/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/94d8f162-daf4-46a1-b8fe-c4889347daa4/Untitled.png)

Not much here, lets look for possible exploits for IIS 6.0

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/41738)

I spent some time trying to get a working shellcode for this exploit but ended up just using Metasploit to exploit this vulnerability.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bb4471a6-624f-46e7-a634-f5fd9035aaf1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bb4471a6-624f-46e7-a634-f5fd9035aaf1/Untitled.png)

Since we have a meterpreter shell we can use the Metasploit built in kernel exploit checker and see what is found.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3526cc79-305f-49f8-8dce-eef6e64d7bdd/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3526cc79-305f-49f8-8dce-eef6e64d7bdd/Untitled.png)

We have a few to try, I went through all of these and got the same error each time.  After I decided to move on I remembered that I should try to migrate processes to upgrade my shell.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f0445d12-aa8f-4000-a3e6-f0e74eb7f386/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f0445d12-aa8f-4000-a3e6-f0e74eb7f386/Untitled.png)

I thought this could be part of the reason I was getting this error and sure enough after I tried again I got an exploit to work!

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edb85c21-c01b-44e3-bb13-4bfb7e9605a2/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edb85c21-c01b-44e3-bb13-4bfb7e9605a2/Untitled.png)
