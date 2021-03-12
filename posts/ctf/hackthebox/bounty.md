---
layout: page
title: Bounty - HackTheBox
permalink: /posts/ctf/hackthebox/bounty.html
---

# Bounty - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.93`

![image](https://user-images.githubusercontent.com/50459517/110980882-f27c5900-832b-11eb-9c0b-2eb095b3288d.png)


Looks like all we have is port 80 open.  Lets check out the website and see what we have.

![image](https://user-images.githubusercontent.com/50459517/110980899-f9a36700-832b-11eb-9b0a-0a6fcbfeffed.png)


Not much here.  I decided to start a directory scan while looking for a public exploit for this version of IIS 7.5.  I found a few possiblities, one was for MS15-034.  A nice article for testing this can be found here.

[https://blog.sucuri.net/2015/04/website-firewall-critical-microsoft-iis-vulnerability-ms15-034.html](https://blog.sucuri.net/2015/04/website-firewall-critical-microsoft-iis-vulnerability-ms15-034.html)

Lets follow this article to test if the server is vulnerable.

![image](https://user-images.githubusercontent.com/50459517/110980956-0b850a00-832c-11eb-8961-cfd1a6b37b2c.png)


It appears that we get the same Error 416 proving this server has not been patched and may be vulnerable.  However after spending some time on this I could not get an RCE out of it, only DoS.  So I decided to move on.

In my directory scan I found a file **transfer.aspx** along with a directory called **uploadedfiles**.  Lets check these out.

![image](https://user-images.githubusercontent.com/50459517/110980980-12138180-832c-11eb-87ea-5e3a598689bf.png)

![image](https://user-images.githubusercontent.com/50459517/110980991-18096280-832c-11eb-8fbb-12ef100e22d2.png)


Looks like we have a place to upload files along with I assume the location the files get uploaded.  We get a Forbidden error on the **uploadedfiles** directory but we still may be able to access the files we uploaded.  Lets try a test .txt file and see what we get.

![image](https://user-images.githubusercontent.com/50459517/110981003-1fc90700-832c-11eb-8c60-33857ccd410a.png)


Hmm looks like we can't upload .txt files?  I assume that means we wouldn't be able to upload .aspx files either.  Lets try to upload multiple different files types to the server and see what we can upload and what we can't.

After some time I found that we can upload jpg, png and config files.  Since we are running IIS 7.5 and have a way of uploading files to server we can upload a file called web.config and have it execute code on the server for us.

[https://poc-server.com/blog/2018/05/22/rce-by-uploading-a-web-config/](https://poc-server.com/blog/2018/05/22/rce-by-uploading-a-web-config/)

I created the web.config file, had it call for Nishang Powershell reverse shell and get a shell.

![image](https://user-images.githubusercontent.com/50459517/110981043-2fe0e680-832c-11eb-8df4-1a27cde4d048.png)

After some enumeration we find that the server may be vulnerable to a few kernel exploits.

![image](https://user-images.githubusercontent.com/50459517/110981069-38392180-832c-11eb-8ab6-fc84c57628e8.png)

I normally have good luck with the MS10-059 kernel exploit so lets run that and see if we can get a shell.

![image](https://user-images.githubusercontent.com/50459517/110981089-3ff8c600-832c-11eb-8b83-4d772afc0cfe.png)

![image](https://user-images.githubusercontent.com/50459517/110981107-45eea700-832c-11eb-986d-e679216dfdfa.png)
