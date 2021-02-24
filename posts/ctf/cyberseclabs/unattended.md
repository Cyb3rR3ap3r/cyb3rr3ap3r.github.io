---
layout: page
title: Unattended - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/unattended.html
---

# Unattended - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.24`

![image](https://user-images.githubusercontent.com/50459517/109046363-45061600-769a-11eb-8b40-c316c61ef07e.png)

Looks like we have a few interesting ports open.  First lets check out HTTP

![image](https://user-images.githubusercontent.com/50459517/109046400-4d5e5100-769a-11eb-9129-6c397a3ca493.png)

It seems to be running HttpFileServer.  I looked up a potential RCE exploit for this version of HttpFileServer

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/39161)

Lets try it out, we need to change the script to use our IP and listening port for the reverse shell.  We also need to host **nc.exe** on port 80 for the script to use. After running a couple times we get a shell back

![image](https://user-images.githubusercontent.com/50459517/109046421-564f2280-769a-11eb-9e8d-90dd9b8ea431.png)

Now that we have a shell, lets copy over WinPEAS and see if we can escalate our privileges.

![image](https://user-images.githubusercontent.com/50459517/109046449-5f3ff400-769a-11eb-8289-a579725bbb67.png)

We see that Unattend.xml is available.  Lets check it out and see if we can find any creds.

![image](https://user-images.githubusercontent.com/50459517/109046485-68c95c00-769a-11eb-8bf3-8becd730d9c7.png)

Nice.  Lets try to log in as Administrator with this password.

![image](https://user-images.githubusercontent.com/50459517/109046516-71219700-769a-11eb-9660-89ed25e03e02.png)

Thats it.  We are now Admin on this machine.
