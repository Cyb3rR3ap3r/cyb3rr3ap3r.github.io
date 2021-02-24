---
layout: page
title: Monitor - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/monitor.html
---

# Monitor - CyberSecLabs
----



First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.21`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9a1923a7-db61-4ddf-b3fc-cf3a5bb07adb/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9a1923a7-db61-4ddf-b3fc-cf3a5bb07adb/Untitled.png)

I first checked out SMB to see if I could find any anonymous shares I could access and found one called **WebBackups**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8802a29f-a3b0-47f4-9477-b50b8bd11d10/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8802a29f-a3b0-47f4-9477-b50b8bd11d10/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/feb86804-2f85-471d-a353-72f3bb0dc45f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/feb86804-2f85-471d-a353-72f3bb0dc45f/Untitled.png)

After getting this file and extracting it, I found an interesting db file. that could contain creds.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f12b87f0-a5c1-4c1b-b572-acce3af18254/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f12b87f0-a5c1-4c1b-b572-acce3af18254/Untitled.png)

I decided to open this database in **sqlitebrowser** which is a nice GUI for browser sqlite db files.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/37834864-b1ae-43a3-a2db-6dae07c627b7/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/37834864-b1ae-43a3-a2db-6dae07c627b7/Untitled.png)

First thing we see is a username and password.  Lets check out the web site and see if we can find a login panel to interact with.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5f1e533b-ae13-4638-b8a1-7d4a197dff32/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5f1e533b-ae13-4638-b8a1-7d4a197dff32/Untitled.png)

Looks like we have PRTG Network Monitor running on the server, lets try 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fc0e870c-eb20-43d9-a982-f68d94b90ae5/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fc0e870c-eb20-43d9-a982-f68d94b90ae5/Untitled.png)

Hmmm, that didn't work..  I continued to browse the database and found a password hash for the user admin.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c328a912-e9de-4433-b47d-d4d404f57c10/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c328a912-e9de-4433-b47d-d4d404f57c10/Untitled.png)

I spent some time trying to crack this but no luck...  I didn't find any other information in the db so decided to search for default creds.  I found that **prtgadmin:prtgadmin** is set by default but that failed as well.  At this point I decided to review the walkthrough for this as I was stuck and figured I was overthinking something, which I was.

The username **prtgadmin** was correct and uses the password we found in the db **Se7vmMqP0al**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/736d48c3-1bee-417d-af91-57973a9c667b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/736d48c3-1bee-417d-af91-57973a9c667b/Untitled.png)

Now that we are logged in, we can look for possible ways to get a shell on this machine.  I found this exploit that looks interesting.

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/46527)

I downloaded and reviewed the usage.  All it needs is the host and the cookie for the admin account.  After that, if vulnerable, it will create a user on the server as **pentest:P3nT3st!** with admin permissions.  Lets try it out and see.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eca08384-eec7-47cb-aa9f-15899fd0fd7a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eca08384-eec7-47cb-aa9f-15899fd0fd7a/Untitled.png)

Looks like it worked!  Now lets try to authenticate with Evil-WinRM

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9351869c-73ad-4737-afb9-bc3046dc6838/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9351869c-73ad-4737-afb9-bc3046dc6838/Untitled.png)

No luck... Now lets try psexec..

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/08dd3e80-ff7a-423c-8fd5-1ab7df74df29/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/08dd3e80-ff7a-423c-8fd5-1ab7df74df29/Untitled.png)

Well, good news is it appears the username:password did in fact get created on the machine but no writable shares so therefore no shell.  This could have been why WinRM failed I'm not sure..

Last chance with RDP, I like to use **remmina** for this.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b350a0ab-0ebe-4133-bf83-154b2e2f2fa4/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b350a0ab-0ebe-4133-bf83-154b2e2f2fa4/Untitled.png)

It worked!  We are in and we are in fact in the administrators group so we have full access to this machine now.
