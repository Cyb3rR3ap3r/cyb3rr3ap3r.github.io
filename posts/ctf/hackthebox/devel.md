---
layout: page
title: Devel - HackTheBox
permalink: /posts/ctf/hackthebox/devel.html
---

# Devel - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.5`

![image](https://user-images.githubusercontent.com/50459517/109317893-d3050c80-7812-11eb-8c85-ccf9be0045b0.png)

Looks like we have 2 ports open.  First thing that I notice is the FTP contents from the nmap results.  It appears to contain a starter page for an IIS server.  Since we have port 80 running IIS, maybe the FTP server is hosting the root directory of the HTTP server.  Lets see if port 80 is hosting a default IIS page.

![image](https://user-images.githubusercontent.com/50459517/109317936-dd270b00-7812-11eb-956a-df1e56f38d88.png)

Great!  Now for a PoC, lets see if we can copy a file over via FTP and execute it via HTTP.  This will help prove if we are dealing with the actually root directory or just a backup.

![image](https://user-images.githubusercontent.com/50459517/109317959-e57f4600-7812-11eb-9901-fe290b341e7b.png)

![image](https://user-images.githubusercontent.com/50459517/109317986-edd78100-7812-11eb-928f-9247d51e2b42.png)

Awesome.  We should be able to upload a malicious .aspx file and execute it to get a shell back.

![image](https://user-images.githubusercontent.com/50459517/109318017-f62fbc00-7812-11eb-9bd5-93264015b783.png)

![image](https://user-images.githubusercontent.com/50459517/109318038-fe87f700-7812-11eb-8e74-2a42c70d4032.png)

![image](https://user-images.githubusercontent.com/50459517/109318071-06479b80-7813-11eb-9770-8444cefd6cd1.png)

I copied over WinPEAS.exe but was unable to get the 32bit version to run correctly..  I decided to run the .bat version but didn't find much so decided to run WinPrivCheck.bat as I like the way it checks for uninstalled patches on the system that may have kernel exploits we could use.

![image](https://user-images.githubusercontent.com/50459517/109318100-0e9fd680-7813-11eb-88f0-55aa6bb89ba2.png)

I looked through a few of these and found that MS11-046 worked and already has a compiled executable here.

[https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)

Just simply copy the executable over and run.

![image](https://user-images.githubusercontent.com/50459517/109318131-1790a800-7813-11eb-95ac-89a25ad4d7fe.png)
