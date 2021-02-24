---
layout: page
title: Unattended - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/unattended.html
---

# Unattended - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.24`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b08fd29-3704-4887-8ba5-1a7a73c16ff9/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b08fd29-3704-4887-8ba5-1a7a73c16ff9/Untitled.png)

First lets check out HTTP

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d01265d4-2a98-4fd0-8819-f1312390df5b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d01265d4-2a98-4fd0-8819-f1312390df5b/Untitled.png)

I looked up a potential RCE exploit for this version of HttpFileServer

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/39161)

Lets try it out, we need to change the script to use our IP and listening port for the reverse shell.  We also need to host **nc.exe** on port 80 for the script to use. After running a couple times we get a shell back

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5c91de6-1661-4799-a63c-24bd2697366b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5c91de6-1661-4799-a63c-24bd2697366b/Untitled.png)

Now that we have a shell, lets copy over WinPEAS and see if we can escalate our privileges.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d046b06f-286c-4787-866a-3ad3e7648cbc/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d046b06f-286c-4787-866a-3ad3e7648cbc/Untitled.png)

We see that Unattend.xml is available.  Lets check it out and see if we can find any creds.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9ec1ad0-434b-4a0d-abee-ca8339418b48/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9ec1ad0-434b-4a0d-abee-ca8339418b48/Untitled.png)

Nice.  Lets try to log in as Administrator with this password.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/53452214-d9ea-46be-b5dc-89d1f335d799/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/53452214-d9ea-46be-b5dc-89d1f335d799/Untitled.png)

Thats it.  We are now Admin on this machine.
