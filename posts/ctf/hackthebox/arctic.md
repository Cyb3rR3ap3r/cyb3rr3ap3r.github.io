---
layout: page
title: Arctic - HackTheBox
permalink: /posts/ctf/hackthebox/arctic.html
---

# Arctic - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.11`

![image](https://user-images.githubusercontent.com/50459517/110968601-59dedc80-831d-11eb-9085-b4882f4bdca5.png)

Hmmm... I got some weird results.  I'm going to assume we have a web server running on port 8500 but not sure.  Lets check it out.

![image](https://user-images.githubusercontent.com/50459517/110968628-62cfae00-831d-11eb-9e87-306b70c5c6c7.png)

Okay, I see CFIDE and it makes me think of ColdFusion.  Lets see if we can get to the default administrator page.

![image](https://user-images.githubusercontent.com/50459517/110968661-6cf1ac80-831d-11eb-9d60-474821007356.png)

Great!  I tried a few default creds but no luck.. I found a nice article on ColdFusion hacking here..

[https://nets.ec/Coldfusion_hacking](https://nets.ec/Coldfusion_hacking)

Following this, it seems that ColdFusion8 is vulnerable to LFI and also you can get the password hash in the **password.properties** file.

![image](https://user-images.githubusercontent.com/50459517/110968779-8abf1180-831d-11eb-9266-eee87c618ac0.png)

Now that we have the password hash we could try to crack the hash, but we can also just bypass the authentication using the guide in the link above.  We just need to paste the hash into the password form, run some javascript that will output another hash, then load that hash into Tamper Data.  This is time sensitive so you have to be quick so to save time you can save the javascript into a bookmark. 

`javascript:alert(hex_hmac_sha1(document.loginform.salt.value,`
`document.loginform.cfadminPassword.value))`

I tired several times to modify the password field via Tamper Data to no luck.  I found better luck modifying with Burp.  Once I completed this I was able to bypass the login.

![image](https://user-images.githubusercontent.com/50459517/110968810-96aad380-831d-11eb-97f9-0c4a94d86deb.png)

Now time to upload a shell.  I used the Scheduled Task to upload a shell onto the server.  I can then run the task and access the new file to get code execution.

![image](https://user-images.githubusercontent.com/50459517/110968846-a2969580-831d-11eb-8b79-bcdd7bd1d6e0.png)

![image](https://user-images.githubusercontent.com/50459517/110968888-acb89400-831d-11eb-8b74-ab130367c5a1.png)

Great!  Now we can run the following command to download NetCat to the machine and get a reverse shell.

![image](https://user-images.githubusercontent.com/50459517/110968932-b7732900-831d-11eb-95cb-c6cf6d36481c.png)

![image](https://user-images.githubusercontent.com/50459517/110968955-bfcb6400-831d-11eb-8e85-9508247c00cb.png)

I ran WinPrivEsc.bat to see if we can find any potential kernel exploits we can use.  The results told us that the MS10-059 patch is not installed!  Lets copy the MS10-059.exe to the machine and see if it works.

![image](https://user-images.githubusercontent.com/50459517/110968980-c8239f00-831d-11eb-844f-c0a3fab63d2e.png)

![image](https://user-images.githubusercontent.com/50459517/110969016-d1147080-831d-11eb-854b-e90754610b9b.png)
