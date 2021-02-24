---
layout: page
title: Monitor - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/monitor.html
---

# Monitor - CyberSecLabs
----



First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.21`

![image](https://user-images.githubusercontent.com/50459517/109044732-5fd78b00-7698-11eb-928c-21e075860aa5.png)

I first checked out SMB to see if I could find any anonymous shares I could access and found one called **WebBackups**

![image](https://user-images.githubusercontent.com/50459517/109044768-6bc34d00-7698-11eb-9ec6-9a65a02653f0.png)

![image](https://user-images.githubusercontent.com/50459517/109044806-754cb500-7698-11eb-838b-c28b3bb2e63d.png)

After getting this file and extracting it, I found an interesting db file. that could contain creds.

![image](https://user-images.githubusercontent.com/50459517/109044829-7e3d8680-7698-11eb-9225-fbe8c212027c.png)

I decided to open this database in **sqlitebrowser** which is a nice GUI for browser sqlite db files.

![image](https://user-images.githubusercontent.com/50459517/109044862-872e5800-7698-11eb-8ce9-2bdd60e864a1.png)

First thing we see is a username and password.  Lets check out the web site and see if we can find a login panel to interact with.

![image](https://user-images.githubusercontent.com/50459517/109044954-a4fbbd00-7698-11eb-888c-2af7ed0ed71c.png)

Looks like we have PRTG Network Monitor running on the server, lets try 

![image](https://user-images.githubusercontent.com/50459517/109044991-ae852500-7698-11eb-987d-e070a8ecb76c.png)

Hmmm, that didn't work..  I continued to browse the database and found a password hash for the user admin.

![image](https://user-images.githubusercontent.com/50459517/109045023-b775f680-7698-11eb-809f-20ca72c9958b.png)

I spent some time trying to crack this but no luck...  I didn't find any other information in the db so decided to search for default creds.  I found that **prtgadmin:prtgadmin** is set by default but that failed as well.  At this point I decided to review the walkthrough for this as I was stuck and figured I was overthinking something, which I was.

The username **prtgadmin** was correct and uses the password we found in the db **Se7vmMqP0al**

![image](https://user-images.githubusercontent.com/50459517/109045060-c197f500-7698-11eb-9c9f-27f3f6b0e5e6.png)

Now that we are logged in, we can look for possible ways to get a shell on this machine.  I found this exploit that looks interesting.

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/46527)

I downloaded and reviewed the usage.  All it needs is the host and the cookie for the admin account.  After that, if vulnerable, it will create a user on the server as **pentest:P3nT3st!** with admin permissions.  Lets try it out and see.

![image](https://user-images.githubusercontent.com/50459517/109045100-cc528a00-7698-11eb-91bf-982790519a0a.png)

Looks like it worked!  Now lets try to authenticate with Evil-WinRM

![image](https://user-images.githubusercontent.com/50459517/109045131-d5dbf200-7698-11eb-895b-8a49e15fad9b.png)

No luck... Now lets try psexec..

![image](https://user-images.githubusercontent.com/50459517/109045163-deccc380-7698-11eb-9a50-fd1463790dd3.png)

Well, good news is it appears the username:password did in fact get created on the machine but no writable shares so therefore no shell.  This could have been why WinRM failed I'm not sure..

Last chance with RDP, I like to use **remmina** for this.

![image](https://user-images.githubusercontent.com/50459517/109045202-e724fe80-7698-11eb-93a8-d948978debd1.png)

It worked!  We are in and we are in fact in the administrators group so we have full access to this machine now.
