---
layout: page
title: Bounty - HackTheBox
permalink: /posts/ctf/hackthebox/bounty.html
---

# Bounty - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.93`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f6833628-5082-420d-b138-d538cadf9cfd/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f6833628-5082-420d-b138-d538cadf9cfd/Untitled.png)

Looks like all we have is port 80 open.  Lets check out the website and see what we have.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d9068995-69f1-47ac-8b3d-8527d27a5399/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d9068995-69f1-47ac-8b3d-8527d27a5399/Untitled.png)

Not much here.  I decided to start a directory scan while looking for a public exploit for this version of IIS 7.5.  I found a few possiblities, one was for MS15-034.  A nice article for testing this can be found here.

[Critical Microsoft IIS Vulnerability Leads to RCE (MS15-034)](https://blog.sucuri.net/2015/04/website-firewall-critical-microsoft-iis-vulnerability-ms15-034.html)

Lets follow this article to test if the server is vulnerable.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c264ce6b-70b9-4f7e-ad78-5c96e982f8b5/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c264ce6b-70b9-4f7e-ad78-5c96e982f8b5/Untitled.png)

It appears that we get the same Error 416 proving this server has not been patched and may be vulnerable.  However after spending some time on this I could not get an RCE out of it, only DoS.  So I decided to move on.

In my directory scan I found a file **transfer.aspx** along with a directory called **uploadedfiles**.  Lets check these out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/478ce68a-1355-4ee4-985e-e942c58aa948/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/478ce68a-1355-4ee4-985e-e942c58aa948/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ac7dec81-2ff0-4902-9aef-9a40703bbfd2/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ac7dec81-2ff0-4902-9aef-9a40703bbfd2/Untitled.png)

Looks like we have a place to upload files along with I assume the location the files get uploaded.  We get a Forbidden error on the **uploadedfiles** directory but we still may be able to access the files we uploaded.  Lets try a test .txt file and see what we get.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bbc54a33-116e-47f0-8764-793b1e720854/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bbc54a33-116e-47f0-8764-793b1e720854/Untitled.png)

Hmm looks like we can't upload .txt files?  I assume that means we wouldn't be able to upload .aspx files either.  Lets try to upload multiple different files types to the server and see what we can upload and what we can't.

After some time I found that we can upload jpg, png and config files.  Since we are running IIS 7.5 and have a way of uploading files to server we can upload a file called web.config and have it execute code on the server for us.

[RCE by uploading a web.config](https://poc-server.com/blog/2018/05/22/rce-by-uploading-a-web-config/)

I created the web.config file, had it call for Nishang Powershell reverse shell and get a shell.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/93f0f1b1-e92f-4208-b9d9-d19c1807d9ef/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/93f0f1b1-e92f-4208-b9d9-d19c1807d9ef/Untitled.png)

After some enumeration we find that the server may be vulnerable to a few kernel exploits.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b78de3b5-a0ca-4e99-a4f1-5ad3fd4e8b51/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b78de3b5-a0ca-4e99-a4f1-5ad3fd4e8b51/Untitled.png)

I normally have good luck with the MS10-059 kernel exploit so lets run that and see if we can get a shell.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d35a1077-8a72-4803-9994-0eae3766d61b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d35a1077-8a72-4803-9994-0eae3766d61b/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/013c13b4-f2b5-4772-96c1-2cd0e3084da8/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/013c13b4-f2b5-4772-96c1-2cd0e3084da8/Untitled.png)
