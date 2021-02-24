---
layout: page
title: Simple - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/simple.html
---

# Simple - CyberSecLabs
----



First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.2`

![image](https://user-images.githubusercontent.com/50459517/109029598-a02f0d00-7688-11eb-9246-79e1db868f4e.png)

Looks like we have 2 ports open.  Lets check out port 80 first.

![image](https://user-images.githubusercontent.com/50459517/109029639-aa510b80-7688-11eb-8056-de9f8ca310e9.png)

Seems to be running **CMS Made Simple** version 2.2.4 so I looked for a possible online exploit and found the following on ExploitDB

![image](https://user-images.githubusercontent.com/50459517/109029681-b341dd00-7688-11eb-8a0b-7de5ef337e04.png)

After downloading and converting to python3 I ran with the following syntax to exploit the SQL Injection and crack the password

`python3 [exploit.py](http://exploit.py/) -u [http://172.31.1.2](http://172.31.1.2/) --crack -w /usr/share/wordlists/rockyou.txt`

![image](https://user-images.githubusercontent.com/50459517/109029724-bc32ae80-7688-11eb-9719-8839dd9c3ba1.png)

I kept getting this error, I tried a few things with the code to no luck..  I think it doesn't like the way one of the lines in the rockyou worklist is encoded, so I just decided to move a different way and brute force since we had the user.  I found the password was **punisher** and logged in.

![image](https://user-images.githubusercontent.com/50459517/109029760-c785da00-7688-11eb-9cbd-7e6e40676445.png)

Looking through the panel, I found a File Manager and it says the current path is **uploads/simplex** so lets try to upload a file and access it via the uploads directory.

![image](https://user-images.githubusercontent.com/50459517/109029790-cfde1500-7688-11eb-804e-5c2b198c74c0.png)

![image](https://user-images.githubusercontent.com/50459517/109029822-d8cee680-7688-11eb-87e0-93dd862c2454.png)

That worked!  Now since this is running Apache in the background lets upload a reverse shell php script and try to get a shell on this machine.  We can use the pentestmonkeys php-reverse-shell for this.

At this point I got stuck trying to find a way to upload the php script.  I tried many different things to bypass whatever filters were in place preventing me from uploading the php scripts.  I looked at other angles to get access to the machine, messing with settings within the panel, trying to edit file names already on the server, etc.  Finally I had to look at the video walkthrough and see what I was missing and noticed he uploaded the php file as **phtml** instead of php.  Tried that and worked perfectly.  Wasted more time then needed on this but I learned so stuff so I guess it wasn't all a waste.

![image](https://user-images.githubusercontent.com/50459517/109029870-e3897b80-7688-11eb-9d46-bf7de90b6bcb.png)

![image](https://user-images.githubusercontent.com/50459517/109029898-ec7a4d00-7688-11eb-8a7d-a3d7feecc6e1.png)

Now lets upload **LinPEAS** and see what it finds.

![image](https://user-images.githubusercontent.com/50459517/109029933-f69c4b80-7688-11eb-83e9-fd0029ea5603.png)

Looks like **/bin/systemctl** has the SUID bit set.  According to GTFOBins, we may be able to abuse this.  I followed the directions on GTFOBins but didn't have much success.

I found more success creating a file called **test.service** and editing it with the values GTFOBins suggests.  I then copied that file over and used **/bin/systemctl** to enable and start the service, resulting in escalation of privilege's 

![image](https://user-images.githubusercontent.com/50459517/109029975-0156e080-7689-11eb-8fa4-791512eda2e8.png)

![image](https://user-images.githubusercontent.com/50459517/109030003-087dee80-7689-11eb-82ec-0f0a8b438457.png)

![image](https://user-images.githubusercontent.com/50459517/109030030-10d62980-7689-11eb-803f-e9d7f5de942c.png)
