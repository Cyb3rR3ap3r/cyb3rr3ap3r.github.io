---
layout: page
title: Nibbles - HackTheBox
permalink: /posts/ctf/hackthebox/nibbles.html
---

# Nibbles - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.75`

![image](https://user-images.githubusercontent.com/50459517/110982159-cd88e580-832d-11eb-8c38-16293c5a9fc6.png)

Looks like we have a couple ports open.  Port 80 looks the most interesting so lets check that out first.

![image](https://user-images.githubusercontent.com/50459517/110982173-d7124d80-832d-11eb-9cfd-6d70a3d487f1.png)

Not much here, Lets look at the source code.

![image](https://user-images.githubusercontent.com/50459517/110982188-dd082e80-832d-11eb-8c70-b7807b87b0e1.png)

Hmmm **/nibbleblog/**  Lets check it out.

![image](https://user-images.githubusercontent.com/50459517/110982223-e42f3c80-832d-11eb-8f81-96211add53b6.png)

Nice.  Looks like we are running Nibbleblog.  I ran a directory fuzzing in the background and found are file called README, lets check it out.

![image](https://user-images.githubusercontent.com/50459517/110982248-ebeee100-832d-11eb-9b0a-80734c5c959a.png)

Looks like we have a version now, lets look up any possible exploits for this version.

[https://curesec.com/blog/article/blog/NibbleBlog-403-Code-Execution-47.html](https://curesec.com/blog/article/blog/NibbleBlog-403-Code-Execution-47.html)

I found this which looks interesting but unfortunately it requires authentication.  It seems like the login panel is at **admin.php** so lets see if we can find it.

![image](https://user-images.githubusercontent.com/50459517/110982303-fad59380-832d-11eb-9237-2c4aac5ec02d.png)

Perfect.  Unfortunately we don't know the password... I tried a few defaults but no luck.  I then decided to try to brute force but.... It blocked me after about 5 attempts, sop I had to reset the box to continue.

![image](https://user-images.githubusercontent.com/50459517/110982325-032dce80-832e-11eb-87db-2e0f92e5336e.png)

While resetting the box I was researching for default creds on Nibbleblog and unfortunately found the password to this box in a writeup... The creds are  **admin:nibbles**

![image](https://user-images.githubusercontent.com/50459517/110982341-088b1900-832e-11eb-9bcf-2bda661decd7.png)

Now that we are authenticated, we should be able to upload a malicious php file to get a shell on this machine.  Just following the above link step by step and it should work just fine.

![image](https://user-images.githubusercontent.com/50459517/110982366-117bea80-832e-11eb-94da-83bcf9433f3c.png)

Recently I have been practicing more post enumeration and persistence activities.  One of the first things I've been doing recently when establishing a shell on a linux machine is the following sequence of commands...

`python3 -c "import pty; pty.spawn('/bin/bash')"      //run on victim's machine`
`CTRL + Z                                            //switches over to your machine`
`stty raw -echo                                      //run on your machine`
`fg                                                  //switches back to victim machine`
`export TERM=xtrm                                    //run on victim machine`

![image](https://user-images.githubusercontent.com/50459517/110982502-31131300-832e-11eb-9b5c-38ba1a9d1300.png)

This enables us to use the arrow keys as well as have tab completion and command history in our reverse shell!  I don't know how I lived in a shell without this for so long!

Now, lets copy over LinPEAS and see what we can find.

![image](https://user-images.githubusercontent.com/50459517/110982480-2b1d3200-832e-11eb-8549-4b0f37f34249.png)

Since we have full access to this file, we can modify it to make a copy of **/bin/bash** to **/tmp** and running as root to get a root shell.  Just add the following into the file and run with sudo.

`cp /bin/bash /tmp/bash; chmod +s /tmp/bash`

![image](https://user-images.githubusercontent.com/50459517/110982527-383a2100-832e-11eb-94b0-bc1b2e88c743.png)

![image](https://user-images.githubusercontent.com/50459517/110982546-3f612f00-832e-11eb-82f2-728d41710690.png)
