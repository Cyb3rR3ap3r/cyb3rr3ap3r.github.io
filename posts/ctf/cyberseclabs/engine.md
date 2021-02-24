---
layout: page
title: Engine - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/engine.html
---

# Engine - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.16`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a286e059-1c9a-4dc9-9598-9c04f46d7bf9/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a286e059-1c9a-4dc9-9598-9c04f46d7bf9/Untitled.png)

Checking out the web server on port 80, we see a default IIS page.  So I started a directory buster and found a directory called **/blog** so lets check that out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/98249ad9-8f47-4b04-aa36-ac098a69aed5/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/98249ad9-8f47-4b04-aa36-ac098a69aed5/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d95deaa0-3fa5-4a69-841f-c13cf8c7bd7b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d95deaa0-3fa5-4a69-841f-c13cf8c7bd7b/Untitled.png)

Looks like we have a blog running **BlogEngine.NET**. I click on the log in button and found the default creds of **admin:admin** was successfully and now I have access to the admin panel.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bfeae591-2cc9-446c-9dc0-1ea990f83162/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bfeae591-2cc9-446c-9dc0-1ea990f83162/Untitled.png)

After spending some time trying to find a way to upload and execute a reverse shell, I found a version of **[BlogEngine.NET](http://blogengine.NET)** so I decided to look into a possible public vulnerability.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9705f6dc-4374-42b3-bfd6-d144083043c4/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9705f6dc-4374-42b3-bfd6-d144083043c4/Untitled.png)

I found a vulnerability that allows for remote code execution via LFI per **CVE-2019-6714** that worked well.

[https://www.exploit-db.com/exploits/46353](https://www.exploit-db.com/exploits/46353)

After following these I was able to get the initial shell onto the machine

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/095b2736-a9d5-44a4-8260-2f0da575e26d/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/095b2736-a9d5-44a4-8260-2f0da575e26d/Untitled.png)

I then uploaded **WinPEAS** to the machine and found some AutoLogin creds for the Administrator account.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4c4f001e-08cb-4def-8a91-8ef428159226/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4c4f001e-08cb-4def-8a91-8ef428159226/Untitled.png)

I used these credentials to log into the machine via WinRM using **evil-winrm**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7dce6ab1-5ded-42db-a73e-7540a62c0b03/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7dce6ab1-5ded-42db-a73e-7540a62c0b03/Untitled.png)
