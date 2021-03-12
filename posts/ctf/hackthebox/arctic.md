---
layout: page
title: Arctic - HackTheBox
permalink: /posts/ctf/hackthebox/arctic.html
---

# Arctic - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.11`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6dd73b2f-adbc-4506-a004-b03916f7eed0/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6dd73b2f-adbc-4506-a004-b03916f7eed0/Untitled.png)

Hmmm... I got some weird results.  I'm going to assume we have a web server running on port 8500 but not sure.  Lets check it out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9990b41f-2f81-445c-9083-12bcecea44a7/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9990b41f-2f81-445c-9083-12bcecea44a7/Untitled.png)

Okay, I see CFIDE and it makes me think of ColdFusion.  Lets see if we can get to the default administrator page.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9446abf6-239b-4cd5-9f72-c5818839676c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9446abf6-239b-4cd5-9f72-c5818839676c/Untitled.png)

Great!  I tried a few default creds but no luck.. I found a nice article on ColdFusion hacking here..

[Coldfusion hacking](https://nets.ec/Coldfusion_hacking)

Following this, it seems that ColdFusion8 is vulnerable to LFI and also you can get the password hash in the **[password.properties](http://password.properties)** file.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a7d78c9a-58a9-4086-92c1-a05e80e0d526/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a7d78c9a-58a9-4086-92c1-a05e80e0d526/Untitled.png)

Now that we have the password hash we could try to crack the hash, but we can also just bypass the authentication using the guide in the link above.  We just need to paste the hash into the password form, run some javascript that will output another hash, then load that hash into Tamper Data.  This is time sensitive so you have to be quick so to save time you can save the javascript into a bookmark. 

`javascript:alert(hex_hmac_sha1(document.loginform.salt.value,document.loginform.cfadminPassword.value))`

I tired several times to modify the password field via Tamper Data to no luck.  I found better luck modifying with Burp.  Once I completed this I was able to bypass the login.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e4617e29-2e70-49a8-8c74-3c56b6cc4c8d/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e4617e29-2e70-49a8-8c74-3c56b6cc4c8d/Untitled.png)

Now time to upload a shell.  I used the Scheduled Task to upload a shell onto the server.  I can then run the task and access the new file to get code execution.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/55eb74fb-c3f5-42f2-bc87-4712dd0e99f6/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/55eb74fb-c3f5-42f2-bc87-4712dd0e99f6/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e8eb9086-63be-4058-849d-b6e4b5c025e6/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e8eb9086-63be-4058-849d-b6e4b5c025e6/Untitled.png)

Great!  Now we can run the following command to download NetCat to the machine and get a reverse shell.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2bcf3188-c65a-4c62-9892-9666de2bf7c2/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2bcf3188-c65a-4c62-9892-9666de2bf7c2/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b5cfff4f-2595-4f58-8c68-b52ac700a2eb/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b5cfff4f-2595-4f58-8c68-b52ac700a2eb/Untitled.png)

I ran WinPrivEsc.bat to see if we can find any potential kernel exploits we can use.  The results told us that the MS10-059 patch is not installed!  Lets copy the MS17-059.exe to the machine and see if it works.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c954b18-9d07-46ce-8e96-c970bfe6c040/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c954b18-9d07-46ce-8e96-c970bfe6c040/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b72ed967-97ad-4273-ae46-416a587190b9/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b72ed967-97ad-4273-ae46-416a587190b9/Untitled.png)
