---
layout: page
title: Simple - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/simple.html
---

# Simple - CyberSecLabs
----



First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.2`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a0a74d1-37ad-4824-aea6-a09c973fb864/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a0a74d1-37ad-4824-aea6-a09c973fb864/Untitled.png)

Looks like we have 2 ports open.  Lets check out port 80 first.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a7415930-4b9c-4efb-8f8e-f8df43cf6f28/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a7415930-4b9c-4efb-8f8e-f8df43cf6f28/Untitled.png)

Seems to be running **CMS Made Simple** version 2.2.4 so I looked for a possible online exploit and found the following on ExploitDB

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/367a2b5c-5e28-40aa-98a8-1a3ee152ae5a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/367a2b5c-5e28-40aa-98a8-1a3ee152ae5a/Untitled.png)

After downloading and converting to python3 I ran with the following syntax to exploit the SQL Injection and crack the password

`python3 [exploit.py](http://exploit.py/) -u [http://172.31.1.2](http://172.31.1.2/) --crack -w /usr/share/wordlists/rockyou.txt`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f5ee0f90-734c-4eac-8a72-ac81d4a2f03c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f5ee0f90-734c-4eac-8a72-ac81d4a2f03c/Untitled.png)

I kept getting this error, I tried a few things with the code to no luck..  I think it doesn't like the way one of the lines in the rockyou worklist is encoded, so I just decided to move a different way and brute force since we had the user.  I found the password was **punisher** and logged in.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c03346fd-0821-4e83-a476-aa302a6c119e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c03346fd-0821-4e83-a476-aa302a6c119e/Untitled.png)

Looking through the panel, I found a File Manager and it says the current path is **uploads/simplex** so lets try to upload a file and access it via the uploads directory.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b7f80ee-5903-4255-ba07-c669f41bc689/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b7f80ee-5903-4255-ba07-c669f41bc689/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1c1d07f9-dc66-496d-82f9-10a0b7c45997/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1c1d07f9-dc66-496d-82f9-10a0b7c45997/Untitled.png)

That worked!  Now since this is running Apache in the background lets upload a reverse shell php script and try to get a shell on this machine.  We can use the pentestmonkeys php-reverse-shell for this.

At this point I got stuck trying to find a way to upload the php script.  I tried many different things to bypass whatever filters were in place preventing me from uploading the php scripts.  I looked at other angles to get access to the machine, messing with settings within the panel, trying to edit file names already on the server, etc.  Finally I had to look at the video walkthrough and see what I was missing and noticed he uploaded the php file as **phtml** instead of php.  Tried that and worked perfectly.  Wasted more time then needed on this but I learned so stuff so I guess it wasn't all a waste.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8b749802-ad3b-4473-9c31-467f31e95521/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8b749802-ad3b-4473-9c31-467f31e95521/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a2ea7d1-15f6-4926-8194-b1e081711680/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a2ea7d1-15f6-4926-8194-b1e081711680/Untitled.png)

Now lets upload **LinPEAS** and see what it finds.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9d75a979-1ae9-498b-aa2f-29bfcb1b6c99/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9d75a979-1ae9-498b-aa2f-29bfcb1b6c99/Untitled.png)

Looks like **/bin/systemctl** has the SUID bit set.  According to GTFOBins, we may be able to abuse this.  I followed the directions on GTFOBins but didn't have much success.

I found more success creating a file called **test.service** and editing it with the values GTFOBins suggests.  I then copied that file over and used **/bin/systemctl** to enable and start the service, resulting in escalation of privilege's 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fa1eff45-78f7-4044-ad5a-5b549478abce/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fa1eff45-78f7-4044-ad5a-5b549478abce/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7c1aecea-b7b3-4cb9-8d38-5b88cad56418/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7c1aecea-b7b3-4cb9-8d38-5b88cad56418/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/13a18e73-bf31-4b66-9105-a2e400024521/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/13a18e73-bf31-4b66-9105-a2e400024521/Untitled.png)
