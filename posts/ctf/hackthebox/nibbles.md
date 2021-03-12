---
layout: page
title: Nibbles - HackTheBox
permalink: /posts/ctf/hackthebox/nibbles.html
---

# Nibbles - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.75`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6cccdc06-7b90-4fb6-af7e-a884f437f895/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6cccdc06-7b90-4fb6-af7e-a884f437f895/Untitled.png)

Looks like we have a couple ports open.  Port 80 looks the most interesting so lets check that out first.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/baef493d-aa32-4c1c-9d04-e2ad39c7b1ad/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/baef493d-aa32-4c1c-9d04-e2ad39c7b1ad/Untitled.png)

Not much here, Lets look at the source code.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d634aa7c-1441-45fd-a84b-9d686f08a996/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d634aa7c-1441-45fd-a84b-9d686f08a996/Untitled.png)

Hmmm **/nibbleblog/**  Lets check it out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6609afeb-8b17-4fb1-a0e7-36f0555742c5/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6609afeb-8b17-4fb1-a0e7-36f0555742c5/Untitled.png)

Nice.  Looks like we are running Nibbleblog.  I ran a directory fuzzing in the background and found are file called README, lets check it out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/afcc6bc2-cada-4f67-a242-2a54b315cb7e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/afcc6bc2-cada-4f67-a242-2a54b315cb7e/Untitled.png)

Looks like we have a version now, lets look up any possible exploits for this version.

[NibbleBlog 4.0.3: Code Execution](https://curesec.com/blog/article/blog/NibbleBlog-403-Code-Execution-47.html)

I found this which looks interesting but unfortunately it requires authentication.  It seems like the login panel is at **admin.php** so lets see if we can find it.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/05596fa0-381c-470b-a6a3-5940601950c7/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/05596fa0-381c-470b-a6a3-5940601950c7/Untitled.png)

Perfect.  Unfortunately we don't know the password... I tried a few defaults but no luck.  I then decided to try to brute force but.... It blocked me after about 5 attempts, sop I had to reset the box to continue.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d15e09d8-06f7-489e-bdd2-573c96e445e4/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d15e09d8-06f7-489e-bdd2-573c96e445e4/Untitled.png)

While resetting the box I was researching for default creds on Nibbleblog and unfortunately found the password to this box in a writeup... The creds are  **admin:nibbles**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/705b966b-408d-4097-a38d-305b583fef4a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/705b966b-408d-4097-a38d-305b583fef4a/Untitled.png)

Now that we are authenticated, we should be able to upload a malicious php file to get a shell on this machine.  Just following the above link step by step and it should work just fine.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f96b3991-7fbd-493f-86cf-da743def6871/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f96b3991-7fbd-493f-86cf-da743def6871/Untitled.png)

Recently I have been practicing more post enumeration and persistence activities.  One of the first things I've been doing recently when establishing a shell on a linux machine is the following sequence of commands...

`python3 -c "import pty; pty.spawn('/bin/bash')"      //run on victim's machine`
`CTRL + Z                                            //switches over to your machine`
`stty raw -echo                                      //run on your machine`
`fg                                                  //switches back to victim machine`
`export TERM=xtrm                                    //run on victim machine`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5579894-f8cf-43b4-88df-c66d14349b1c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5579894-f8cf-43b4-88df-c66d14349b1c/Untitled.png)

This enables us to use the arrow keys as well as have tab completion and command history in our reverse shell!  I don't know how I lived in a shell without this for so long!

Now, lets copy over LinPEAS and see what we can find.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8653afec-380f-4607-8a7c-2159bd9816f7/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8653afec-380f-4607-8a7c-2159bd9816f7/Untitled.png)

Since we have full access to this file, we can modify it to make a copy of **/bin/bash** to **/tmp** and running as root to get a root shell.  Just add the following into the file and run with sudo.

`cp /bin/bash /tmp/bash; chmod +s /tmp/bash`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f206842b-86f4-4427-abd2-c039049c4337/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f206842b-86f4-4427-abd2-c039049c4337/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7d701aba-7eed-49da-bab4-11cee278dae0/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7d701aba-7eed-49da-bab4-11cee278dae0/Untitled.png)
