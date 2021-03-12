---
layout: page
title: Grandpa - HackTheBox
permalink: /posts/ctf/hackthebox/grandpa.html
---

# Grandpa - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.13`

![image](https://user-images.githubusercontent.com/50459517/110972337-81d03f00-8321-11eb-86f7-00be2a0757e3.png)

Looks like we only have port 80 to deal with, lets check it out.

![image](https://user-images.githubusercontent.com/50459517/110972379-8c8ad400-8321-11eb-980c-00458c1a5dee.png)

Not much here, lets look for possible exploits for IIS 6.0

[https://www.exploit-db.com/exploits/41738](https://www.exploit-db.com/exploits/41738)

I spent some time trying to get a working shellcode for this exploit but ended up just using Metasploit to exploit this vulnerability.

![image](https://user-images.githubusercontent.com/50459517/110972413-97456900-8321-11eb-9bfe-71bb4288bf7b.png)

Since we have a meterpreter shell we can use the Metasploit built in kernel exploit checker and see what is found.

![image](https://user-images.githubusercontent.com/50459517/110972490-a9bfa280-8321-11eb-9cc0-2a1cd1d8a56e.png)

We have a few to try, I went through all of these and got the same error each time.  After I decided to move on I remembered that I should try to migrate processes to upgrade my shell.

![image](https://user-images.githubusercontent.com/50459517/110972534-b47a3780-8321-11eb-8ac2-4ade570b99d9.png)

I thought this could be part of the reason I was getting this error and sure enough after I tried again I got an exploit to work!

![image](https://user-images.githubusercontent.com/50459517/110972565-bd6b0900-8321-11eb-86ef-6d3ec72c502f.png)
