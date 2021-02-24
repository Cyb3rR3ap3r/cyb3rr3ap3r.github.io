---
layout: page
title: Engine - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/engine.html
---

# Engine - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.16`

![image](https://user-images.githubusercontent.com/50459517/109043545-13d81680-7697-11eb-8759-8edfeb3c1636.png)

Checking out the web server on port 80, we see a default IIS page.  So I started a directory buster and found a directory called **/blog** so lets check that out.

![image](https://user-images.githubusercontent.com/50459517/109043587-1e92ab80-7697-11eb-85e2-447ee578fbd8.png)

![image](https://user-images.githubusercontent.com/50459517/109044231-c5774780-7697-11eb-88cf-4cbd030425eb.png)

Looks like we have a blog running **BlogEngine.NET**. I click on the log in button and found the default creds of **admin:admin** was successfully and now I have access to the admin panel.

![image](https://user-images.githubusercontent.com/50459517/109043699-3833f300-7697-11eb-808c-6f7995773a78.png)

After spending some time trying to find a way to upload and execute a reverse shell, I found a version of BlogEngine.NET so I decided to look into a possible public vulnerability.

![image](https://user-images.githubusercontent.com/50459517/109043745-44b84b80-7697-11eb-8f0a-d03ba86e304c.png)

I found a vulnerability that allows for remote code execution via LFI per **CVE-2019-6714** that worked well.

[https://www.exploit-db.com/exploits/46353](https://www.exploit-db.com/exploits/46353)

After following these I was able to get the initial shell onto the machine

![image](https://user-images.githubusercontent.com/50459517/109043793-513ca400-7697-11eb-8fea-b9f46f6cdd8a.png)

I then uploaded **WinPEAS** to the machine and found some AutoLogin creds for the Administrator account.

![image](https://user-images.githubusercontent.com/50459517/109043847-5d286600-7697-11eb-9209-d4f3a272e39d.png)

I used these credentials to log into the machine via WinRM using **evil-winrm**

![image](https://user-images.githubusercontent.com/50459517/109043895-674a6480-7697-11eb-9300-98f33b7bee74.png)
